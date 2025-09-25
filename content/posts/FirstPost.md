+++
date = '2025-03-22T22:29:11+05:30'
draft = true
title = 'Hilt Snippets'
+++


#### Define  a  Custom component
```kt{linenos=inline}
@CustomScope // Component specific scope
@DefineComponent(parent = SingletonComponent::class) // component should be direct or indirect child of SingletonComponent
interface CustomComponent {

    @DefineComponent.Builder
    interface Builder {
        fun build(): CustomComponent
    }
}
```
---


#### Build Custom Component
```kt{linenos=inline}
class CustomComponentManager @Inject constructor(
    private val customComponentProvider: Provider<CustomComponent.Builder>
){
    var customComponent = customComponentProvider.get().build()

}
```
---
