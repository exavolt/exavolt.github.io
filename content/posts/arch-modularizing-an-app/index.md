+++
title = 'Modularizing an app'
date = 2023-12-18T16:50:40Z
draft = false
lang = 'en-US'
tags = ['architecture', 'modular']
+++

Why do we want modularize our apps? Jump to the last section of this article for the rationale.

---

Imagine we have a two app projects. One of them is a task management app (`tasker`) while the other is a chat app (`messenger`).

We have this for the `tasker` app:

```
.
├── models
│   ├── user_model.dart
│   └── task_model.dart
├── repositories
│   ├── user_repository.dart
│   └── task_repository.dart
├── views
│   ├── login_view.dart
│   └── task_view.dart
└── services
    ├── auth_service.dart
    └── task_service.dart
```

While the `messenger` app goes like this:
```
.
├── models
│   ├── user_model.dart
│   └── chat_model.dart
├── repositories
│   ├── user_repository.dart
│   └── chat_repository.dart
├── views
│   ├── login_view.dart
│   └── chat_view.dart
└── services
    ├── auth_service.dart
    └── chat_service.dart
```

Both of these projects are NOT [modular](https://en.wikipedia.org/wiki/Modular_programming). The reason is because they group together various business into folders, e.g., in the folder `repositories` we can find `user_repository` and `chat_repository` which both are coming from two different, unrelated business domains.

---

To make them modular, we must first map the business domains so that we can draw [the boundaries between various concerns](https://en.wikipedia.org/wiki/Separation_of_concerns) (coordinate with product manager or product owner so that the domain mapping will match). Each module will have a **single responsibilty** for a single concern or a single business domain (it can be hierarchical though, e.g., auth module can be split into smaller sub-modules, e.g., `oauth2`, `webauthn`).

> A program that embodies separation-of-concerns well is called a modular program.

We can start by things that are common in above projects. Looking at the both apps, we can see that they share the same concern, which is the authentication and authorization (auth). The usual way to implement auth in different apps like above would most likely through [copy-paste](https://en.wikipedia.org/wiki/Copy-and-paste_programming) as the business around auth across apps are roughly the same.

> Modularity, and hence separation of concerns, is achieved by encapsulating information inside a section of code that has a well-defined interface.

The parts of the apps other than the auth module don't need to know the details about how the auth works, which remote API it uses, how and where the user data related to auth is stored, session management (refresh token), etc. We can hide these details. We just need to define a well-defined interface so that other part of the apps to access information managed by the auth module. Examples of the interface would be to check whether a user is signed in and to get information about signed in user.

Thus, we can put all business related to auth into its own module, e.g., `auth`. The `auth` module should contain everything related to auth, that includes data structures, services, views, and assets.

```
.
└── auth                            # the module. Contains everything auth-related.
    ├── models                      # more like entities. Contains all auth-related entities.
    │   └── auth_user_model.dart    # from models/user_model.dart. the primary entity in auth is User
    ├── views                       # Contains all auth-related views.
    │   ├── auth_login_view.dart
    │   └── auth_signup_view.dart
    └── services                    # Contains all auth-related services (business logics).
        ├── repositories
        │   └── auth_user_repository.dart
        └── auth_service.dart       # This is the entry point for other modules to integrate with auth service
```

If we look at the structure above, it's like a module is a small app. Well, we can think it like that. Read on for more about this.

By putting everything into a single package (folder), we have turned it into a reusable module. We can even put this module into a repository or package it and publish it somewhere so it can be reused by others.

The key here is that `auth` module must not contain other domain's business, e.g., it must not contain things like assigned tasks or chat messages.

Applying it to the `tasker` project:

```
.
├── app         # app's entry point and as the integrator module
│   ├── views
│   │   └── app_view.dart
│   └── services
│       └── app_service.dart
├── auth        # Our auth module. It could be inlined in the project, use import redirection, symlink, or git submodule, or import the published package
└── tasker      # Here, we've turned the app's main feature as a module
    ├── models
    │   ├── tasker_task_model.dart
    ├── views
    │   ├── tasker_task_list_view.dart
    │   └── tasker_task_detail_view.dart
    └── services
        ├── repositories
        │   └── tasker_task_repository.dart
        └── tasker_service.dart
```

The same can be applied to the messenger app project. In fact, we can combine it with the `tasker` module without ~~creating chaos~~ increasing the complexity by much. This is how we create a "superapp", i.e, an app that comprise of smaller "apps":

```
.
├── app         # the superapp (main module)
│   ├── views
│   │   ├── app_home_page.dart
│   │   └── app_view.dart
│   └── services
│       └── app_service.dart
├── auth        # core module
├── messenger   # feature / business module
├── tasker      # feature / business module
...
└── <more modules>
```

---

With a proper modularization, we can just build various modules for different business domains, and then, depending on the demand or direction, we can just combine these (reusable) modules to build application(s). It means that modularization could **facilitate development agility**.

In a very large app, each module can be assigned to different developers, tribes / squads which are usually already split by business domains. A squad is usually responsible for one or more modules based on its domain(s) which means **feature ownerships are more indicative** through project structure. This is related with the next point.

By putting each business domain into its own folder / package, it should make it easier to detect cross-domain business to prevent blurring the separation of concerns. For example, I want to implement user story in the messenger app. We might thought to update user data in `auth` to add story attribute. But as story is not related to auth, it's better to create a new module that handles stories, and keep the `auth` module focused on authentication and authorization. If we are strict on respecting concern boundaries like this, it would **increase our codebase maintainability**.

Through modularization, it makes it easier to minimize the interface (interface here is API -- package API and interprocess API) exposed by our modules. **Smaller, well-defined interface means less bugs, less maintenance burden, more maintainability**.

While the example is for Flutter apps, the same modularization can be applied to other languages and frameworks, including for backend services. Modularization in backend services would make it **easier to transition from monolithic to microservices**.
