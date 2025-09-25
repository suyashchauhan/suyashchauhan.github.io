+++
date = '2025-06-19T21:12:21+05:30'
draft = true
title = 'Compose'
+++

## Thinking in Compose 
- declarative API that allows you to render your app UI without imperatively mutating frontend views

- Manipulating views manually increases the likelihood of errors
- 

## Managing State

Composition: a description of the UI built by Jetpack Compose when it executes composables.

Initial composition: creation of a Composition by running composables the first time.

Recomposition: re-running composables to update the Composition when data changes.

A value computed by **remember** is stored in the Composition during initial composition, and the stored value is returned during recomposition. 

While remember helps you retain state across recompositions, the state is  **NOT** retained across configuration changes.

Mutable objects that are not observable, such as ArrayList or a mutable data class, are not observable by Compose and don't trigger a recomposition when they change.


Before reading another observable type in Compose, you must convert it to a State<T> so that composables can automatically recompose when the state changes.

### Navigation in compose 

- NavHost
- navController 
- Navigation Graph 