---
title: managing authentication with react context
date: 2019-09-22 22:32:18
tags:
---

Last week i posted about choosing a CSS framework where i built the initial index UI, the following day(s) i started working on the Login and Register Component and several server side functionality including authentication.

## Handling Authentication with Context API
React's context API takes the clutter away of passing mandatory props, that are needed by every component, down your whole component tree. Most often components in between are not interested in these props and this adds a lot of complexity to the application. 
Goal: Show a dynamic buttons in the header and manipulate component behaviour based on the AuthService state. 

I created an initial context data `userData` `isAuthenticated` and `loginPopUp`
You can see that i have some functions that checks the validity of the session and react to it when the component mounts.

```
import React, { createContext } from "react";

export const AuthServiceContext = createContext({
  userData: null,
  isAuthenticated: false,
  loginPopUp: false,
});

export class AuthService extends React.Component {
  componentDidMount() {
    const { history } = this.props;
    if (!this._validSession()) {
      this._logOut();
    } else {
      try {
        const userData = this._getUserData();
        this.setState({
          userData,
          isAuthenticated: true,
        });
      } catch (err) {
        this._logOut();
        history.push("/");
      }
    }
  }

  _getUserData() {
    const token = localStorage.getItem("token");
    const data = decode(token);
    return data;
  }

  _validSession = () => {
    const token = this._getToken();
    return !!token && !this.isTokenExpired(token); 
  };


  state = {
    userData: [],
    isAuthenticated: false,
    loginPopUp: false,
  };

  render() {
    return (
      <AuthServiceContext.Provider value={this.state}>
        {this.props.children}
      </AuthServiceContext.Provider>
    );
  }
}

```
We can now wrapped our consuming components with this Service.
```
function App(){
  return(
    <AuthService>
        <Header/>
        <Moda/>
    </AuthService>
  )
}
```

There are different ways to consume the context using the `Class.contextTpe` API for classes and `useContext` hook for functional component or by exposing the context consumer component.

I used the `useContext` hook since the consumers are both functional component. I imported the `AuthService` and told react to use this context and take some of its exposed values with object destructuring.

This is what the `Header` looks like. 

```
import React, { useContext } from "react";
import { AuthServiceContext } from "../../utils/index";
function HeaderComponent(props) {
  const { isAuthenticated, _logOut, setModalShow } = useContext(AuthServiceContext);
  return (
    <nav className="navbar" role="navigation" aria-label="main navigation">
      <div className="navbar-brand">
        <a className="navbar-item" href="/">
          <div className="header-logo"></div>
        </a>

        <a
          role="button"
          className="navbar-burger burger"
          aria-label="menu"
          aria-expanded="false"
          data-target="navbarBasicExample"
        >
          <span aria-hidden="true"></span>
          <span aria-hidden="true"></span>
          <span aria-hidden="true"></span>
        </a>
      </div>

      <div className="navbar-menu">
        <div className="navbar-start">
          {isAuthenticated ? (
            <>
              <a className="navbar-item" onClick={() => _logOut()}>
                Log out
              </a>
              <Link to="/dashboard" className="navbar-item">
                Dashboard
              </Link>
            </>
          ) : (
              <a className="navbar-item" onClick={() => setModalShow(true)}>
                Sign in
            </a>
            )}
        </div>
        <div className="navbar-end">
          <Link to="/list" className="navbar-item">
            List a property
          </Link>
          <a className="navbar-item">Advertise with us</a>
        </div>
      </div>
    </nav>
  );
}

export default HeaderComponent;
```

Same thing goes with the `Modal`. I am also using the `useState` hook to handle which forms to show.
```
import React, { useContext, useState } from "react";
import { AuthServiceContext } from "../../utils/index";

export function Modal(props) {
  const authService = useContext(AuthServiceContext);
  const [defaultForm, setDefaultForm] = useState(true);

  const showForm = defaultForm ? (
    <Login contextProps={[defaultForm, setDefaultForm]} />
  ) : (
      <Register contextProps={[defaultForm, setDefaultForm]} />
    );



  return (
    <>
      {authService.loginPopUp ? (
        <div className="modal is-active">
          <div
            className="modal-background"
            onClick={() => authService.setModalShow(false)}
          ></div>
          <div className="modal-content">{showForm}</div>
          <button
            className="modal-close is-large"
            aria-label="close"
            onClick={() => authService.setModalShow(false)}
          ></button>
        </div>
      ) : null}
    </>
  );
}
```
That's it! were done!
The only problem i have with Context API is the overuse of wrapping components.. We can fix this with using a HoC but i'll leave it for now.

The final product feature
<ul>
  <li>A modal that popups if a user is not authenticated</li>
  <li>A modal with dynamic form</li>
  <li>A hader with dynamic buttons based on context states</li>
</ul>

![final output](/images/ezgif.com-video-to-gif.gif)

