---
layout: post
title:  "A simple React Router"
date:   2017-06-28 16:25:00 -0400
categories: react redux routing hacking
---

During our last React / Redux project, we started to take our existing single page, single component app and wanted to add some more URL based routing.  We had done some hacky stuff earlier in the project but had a need to make our URL parameter parsing real.  So, we took at look at some of the options out there.

We liked the [react-router](https://github.com/ReactTraining/react-router) project because it is widely used and seemed like a good bridge between the URL and the react props.  But, since we were using redux, we had state stored in redux.  We also had components that needed to either render differently or container components that needed to render different children based on what the navigation state was.  There are some projects designed to help the integration of react-router and redux, however, they seemed either hacky (because they never guaranteed when the redux store would be updated) or super complex (requiring lots of boilerplate and hard to grok).  We decided to create a few components to bridge the gap.

We wanted the data flow to look like this:
1. User (or code) changes URL parameters 
2. Trigger React router render
3. dispatch Redux action
4. navigation state change in the navigation reducer
5. components react to navigationstate change 

We also wanted code that fit in with our existing architecture and was easy for us to understand and reason about.

First, we created some navigation state that components can inspect and making rendering decisions based on the navigation parameters.

````typescript
interface NavigationState {
  basePath: string;
  projectSlug: string;
  taskSlug: string;
  userTasksSlug: string;
}
````

In our navigationActions.ts, we had two types of actions.  One is the action that reacts to changes in the navigation.  The other is an action that you dispatch when you want to trigger a URL parameter or query string change.

````typescript

// any connected component can navigate to my tasks or the first project.
export const navigateToFirstProject = asyncActionCreator(() => {
  return (dispatch, getState) => {
    navigationUtils.navigateToProject(projectSelectors.getFirstProject(getState()));
  };
});

export const navigateToMyTasks = asyncActionCreator(() => {
  return (dispatch, getState) => {
    navigationUtils.navigateToMyTasks();
  };
});

// this is called from code close to react-router
export const navigationChanged = asyncActionCreator<{basePath: string, lastBasePath: string, projectSlug: string, taskSlug: string, userTasksSlug: string}>(payload => {
  const { basePath, projectSlug } = payload;

  return (dispatch, getState) => {
    if (basePath === navigationUtils.PROJECTS_BASE) {
      // Handle some edge cases where we need to change the URL to a valid path
      const selectedProjectId = projectSlug;
      if (isInvalidProject(selectedProjectId, getState())) {
        dispatch(showInvalidProjectModal());
      }

      if (!selectedProjectId) {
        dispatch(navigateToFirstProject());
      }
    }
    // This action will trigger the navigation state update
    dispatch(doNavigationChanged(payload));
  };
});
````

In this case, if I am in a connected component and want to navigate to "My Tasks", I would dispatch navigateToMyTasks.  This means I can still operate on actions to change state and let the action code be responsible for how the navigation will work.

navigationUtils is where the URL construction happens.  This becomes the only place that the browserHistory API is intefaced with and keeps the rest of the applicaiton working with state values, rather than parsing URLs.  It also encapsulates constants for path parts.


````typescript
export const navigateToProjectId = (id: string) => {
  if (!id) {
    const currentLocation = (browserHistory as any).getCurrentLocation();
    if (currentLocation && currentLocation.pathname !== entity.getRootUrl() ) {
      browserHistory.push(entity.getRootUrl());
    }
    return;
  }
  const slug = entity.getUrlForProjectId(id);
  browserHistory.push(slug);
};

export const navigateToTaskId = (taskId: string, projectId: string) => {
  if ( !taskId ) {
    navigateToProjectId(projectId);
    return;
  }
  const slug = entity.getUrlForTaskId(taskId, projectId);
  const currentLocation = (browserHistory as any).getCurrentLocation();
  if (slug !== currentLocation ) {
    browserHistory.push(slug);
  }
};

````

In our App.tsx, we created paths and routes using react-router.  Here, we use a new connected component called RouterWrapper that takes the props that react-router passes (params, location).  Many apps will not do this - they will have separate top level components for different routing.  However, we compose our views from components below these layers so we use the same top level component (RouterWrapper) for all routes.

````typescript
  const App = require('./app/RouterWrapper').default;
    ReactDOM.render(
    <MuiThemeProvider muiTheme={ getMuiTheme(hotTheme) }>
      <Provider store={ store }>
        <Router history={browserHistory}>
          <Route path={`${CLIENT_SIDE_ROOT}`} component={App} >
            <Route path="hubs/:projectSlug(/)" component={App} >
              <Route path="(tasks/:taskSlug)" component={App} />
            </Route>
            <Route path="userTasks/:userTasksSlug(/)" component={App} />
            <Route path="gettingStarted" component={App} />
          </Route>
          <Route path={`${CLIENT_SIDE_ROOT}*`} component={App} />
        </Router>
      </Provider>
    </MuiThemeProvider>,
    rootEl);
````
Then in RouterWrapper.tsx, we use the fact that we have a connected redux component and in the fetchData we can access the params props passed from ReactRouter in the above snippet.  This code is then reponsible for dispatching the navigationChanged action with all of the parameters from react-router.

````typescript
const fetchData = (props: RouterWrapperProps) => {
  if (!props.params.projectSlug && props.location.pathname.includes(navigationUtils.PROJECTS_BASE)) {
    props.navigateToFirstProject();
  }
  setNavigationChanged(props);
};

const setNavigationChanged = (props: RouterWrapperProps) => {
  return props.navigationChanged({basePath: navigationUtils.getBasePath(props.location.pathname), lastBasePath: props.navigationState.basePath, projectSlug: props.params.projectSlug, taskSlug: props.params.taskSlug, userTasksSlug: props.params.userTasksSlug});
};

export class RouterWrapper extends GriffinComponent<RouterWrapperProps, {}> {
  render() {
    return <MainView navigationState={this.props.navigationState} />;
  }
};

export default (connect<RouterWrapperStateProps, RouterWrapperDispatchProps, RouterWrapperOwnProps>(mapStateToProps, mapDispatchToProps)(fetch<RouterWrapperProps>(fetchData)(RouterWrapper)));

````

After all of these changes, we have a complete routing solution that gives us one way data flow (we only react to url changes as actions) and lets react-router do its job, while keeping all of our redux components blissfully unaware of how the routing logic works.

I'm keeping an eye on [react router redux](https://github.com/ReactTraining/react-router/tree/master/packages/react-router-redux) because as redux has taken over as the de factor React flux implementation, other libraries are moving to becoming redux aware or redux native.  It's possible we can use off the shelf next time, but our simple approach gave us clarity that we didn't get out of the box with the third party libraries that existed when we were making our decision to enhance our routing code.
