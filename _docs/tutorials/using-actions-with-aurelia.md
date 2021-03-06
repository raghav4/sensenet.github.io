---
title: "Using sensenet actions with the client packages and Aurelia Framework"
category: Tutorials
index: 1
version: v7.0
tags: [aurelia, client packages, typescript, actions, step by step, delete]
description: Step-by-step tutorial with sensenet actions using the client packages and Aurelia framework
---

# Using sensenet actions with the client packages and Aurelia Framework

Custom OData actions can be useful if you want to execute a bulk action, implement transaction logic or want to use elevation. You can also execute them in an easy and straightforward way using the sensenet client packages.

## A simple example: deleting a content

As we know [how to start working with sensenet on the frontend with the client packages and aurelia](/docs/tutorials/starting-out-with-aurelia/), we can continue improving our example application by calling a simple OData action - deleting a content.

### Updating a template

As the fist step we modify the template of the *content-list* component and add a button for the delete action:

```html

<template>
  <div>
    <input readonly value.bind="currentContent.Path" />
    <ul>
      <li>
        <a click.delegate="navigateUp()">[..]</a>
      </li>
      <li repeat.for="child of children">
        <a click.delegate='navigate(child)'>${child.DisplayName || child.Name}</a>
        <button click.delegate="delete(child)">Delete</button>
      </li>
    </ul>
  </div>
</template>
```

### Calling the Delete action

Calling a sensenet action is easy: you have to call and *await* an async method. We can do that in the view-model of the *content-list* component, adding the following *delete()* method:

```ts
async delete(item: GenericContent) {
    // ask for confirmation
    if (confirm(`Really delete ${item.DisplayName || item.Name}?`)){
        // execute the delete action
        const deleteResponse = await this.repository.delete({
            idOrPath: item.Path,
            permanent: false,
        });
        // reload the children
        this.currentContentChanged(this.currentContent);
    }
}
```
The method will be called by clicking the delete button. It will ask for confirmation, execute the delete action and trigger a reload.

## Execute a custom OData action

Since sensenet is a highly customizable development platform, you can [create](/docs/tutorials/how-to-create-a-custom-odata-action/) custom OData actions.

Usually an action has a name, a method type (GET or POST if it modifies data), a content in the repository as context, specified *parameters* and a specified *response type*.

There is an API endpoint for calling custom actions in the [core](https://www.npmjs.com/package/@sensenet/client-core) client package called [executeAction](/api/@sensenet/client-core/classes/repository.html#executeaction)

If you are using Typescript you can call your actions using generic parameters: ``TBodyType`` is the type definition for the data you should post and ``TReturns`` will be the response model.

You can also create a wrapper method for your actions - that can be responsible for parameter parsing, content type tightening or just simplify your public API. The following example shows how you can wrap your custom action called "MyAction" that is accessible on User contents and returns a collection of users.

```ts
interface MyActionBody {
    myActionParameter: string
}

const myCustomUserAction = (user: User, myActionBody: MyActionBody, oDataOptions: IODataParams<User>) =>
    myRepository.executeAction<MyActionBody, IODataCollectionResponse<User>>({
        idOrPath: user.Id || user.Path,
        method: 'GET',
        name: 'MyAction',
        body: myActionBody,
        oDataOptions,
    })
```

Now you can simply call and await ``myCustomUserAction`` method to trigger your action.

### Actions in the client packages

The @sensenet/client-core package contains shortcuts for common actions like copy, move or delete, [security](/api/@sensenet/client-core/classes/security.html) and [versioning](/api/@sensenet/client-core/classes/versioning.html) related actions.

You can also use actions with React and Redux, there are already some posts and tutorials [here](/blog/2017/09/20/refactoring-sensenet-redux-store) and [here](/docs/tutorials/working-with-actions-in-sensenet-using-reactjs/) about this topic.
