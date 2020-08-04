---
title: "User Identity and Management Kratos & React"
published: false
description: "Implement user identity and management in a React application with ORY Kratos, an open source solution for user login, registration, two factor authentication and profile management."
technologies:
  - { label: "ORY Kratos", url: "https://github.com/ory/kratos" }
  - { label: "React", url: "https://github.com/facebook/react" }
---
- **What is Kratos**

[Kratos](https://github.com/ory/kratos) is an open source identity and user management solution. It covers a set of common use case scenarios such as login, registration and profile management. Kratos is API driven, meaning that it can be used across most application types such as single page applications and server-like applications.

- **Features**

Kratos includes many typical features out of the box:

- Login (including social sign on)
- Registration
- Two Factor Authentication
- Account Verification
- Account Recovery
- Flexible Profile & Account Management
- Public and Admin APIs
- SDKs for a handful of popular languages

- **When To Use Kratos**

- You need control over the UI layer
- You need a self hosted solution
- You need a simple and scalable user and identity management system

- Scale

- Easy to deploy: 
  - Go binary or Docker image
  - Compiled binary has no system or library dependencies
  - Less than 20MB
- Designed to be scaled horizontally with container orchestration technologies such Docker and Kubernetes

- Notes

- Written in Go
- Requires a relational database, currently SQLite, PostgreSQL, MySQL and CockroachDB are supported
  - Hosted solution such as Google Cloud or AWS is recommended 

- Pros

- You have complete control over the user UI
- Small and concise public and admin API
  - Won't take long to get a working prototype running
- Author's reply to issues quickly

- Cons

- Risk: 
  - Small project
  - Project has 1,2000 stars at the time of writing
  - Big players using ORY products in production such as Segment.io and Arduino
- You have to manage your own infrastructure and data
- No solution currently for refreshing sessions without a redirect
- No hosted solution

- Hooks aren't approachable with JavaScript

- Application

