## Evolvable APIs

As our development teams grow, so too does the importance of strong API boundaries. These boundaries, however, do not need to be set in stone. Over time they can grow to incorporate new functionality without impeding existing users of these APIs. This post will outline some decisions that can be made upfront which help maximize the possibility of compatible changes[1] in the future.

The programming language used here is not real; it is meant to be illustrative and hopefully familiar. Unfortunately, the actual programming language you use will determine the extent to which you are able to support compatible changes - so this post also presents an alternative that might be helpful. Keep in mind that the recommendations here are not always going to be applicable: Only use them if they solve an actual problem for you.

## Starting from the Bottom
Let's consider the case of creating the API for a `Button` component in a user interface. To help fully illustrate the properties of the ideal (in my opinion) state of such an API, I'll begin by showing some less than ideal examples.

Poor API design typically results in buttons that look like this:
```kotlin
Button(
    "Download",
    "Download",
    Color.green,
    true,
    null,
    download
)
```
Why is the `"Download"` string duplicated? What is being set to `true`, or `null`. What is `download` signifying? There are many questions that come to mind when seeing a component like this. The biggest issue at play here is that the arguments mean very little to the reader in this context. The reader needs to refer to some external source of information to read this code.

One simple language feature which immediately improves the readability is to use named parameters:
```kotlin
Button(
    text = "Download",
    textWhenDisabled = "Download",
    backgroundColor = Color.green,
    enabled = true,
    onHover = null,
    onClick = download
)
```
It is not much more clear what the arguments mean. The `"Download"` string is actually being set for two different states, the boolean signifies that the button is enabled, the `null` signifies that nothing should happen when hovering over the button, and the button should trigger some sort of download when clicked.

This is better, but it's too noisy! Why are we defining a value for `onHover` when it is not of interest to us, and surely the default state of a button should be enabled. To get around issues like this, we can design our API with sensible default values. Many languages will let us define these right in the definition of the button:

```kotlin
Button(text = "Download", onClick = download)
```

The user can now simply leave out the arguments that they don't care about. This results in terse usages of the button, while keeping the API flexible.

There are more benefits to this approach which are perhaps less immediately obvious. When new functionality is added to the button, existing callers do not need to be updated as long as a sensible default value can be provided. When using named arguments, the order of the arguments is also irrelevant, which really helps reduce cognitive load. Unfortunately, some languages still require parameters to be defined in order even when they are given names (e.g. Swift). One convention to address this is to add parameters with default values at the end of the parameter list.

Our API is already looking good! New functionality can be added without breaking existing callers, and usages of the API are clear and concise. I believe there is one more step that can be taken: using heterogenous maps for the arguments!
```kotlin
Button({ text: "Download", onClick: download })
```

The difference here is subtle: instead of passing the arguments directly to the button, they are packaged up in a single object (i.e. map, dictionary, hash, or whatever your language supports). This enables the creation of generic, higher-order components which can transform any component built with this pattern. [React.js](https://reactjs.org/) is a great example of the power of this pattern. 

Structurally typed langauges (like Typescript) or dynamic languages like Clojure can make great use of this last suggestion. However, nominally typed languages often require large amounts of boilerplate to adopt it and often force rigidity in the implementation since arguments cannot be removed without breaking existing callers. Languages (e.g. Java or Objective-C) which don't have great support for first class data literals are also a poor fit for this last suggestion.

## Alternative: Builder Pattern

If the language you are using does not support named arguments, default values, or heterogenous maps as described above, there are some other patterns you can try. A notable one is the builder pattern, in which you turn each argument into its own function call. Each of these functions returns a modified instance of the original object:
```kotlin
Button()
  .text("Download")
  .onClick(download)
  // Optional values can be specified with additional function calls
  .backgroundColor(Color.green)
  .isEnabled(true)
```
This approach gives you many of the benefits of named arguments with default values. Unfortunately it tends to require signficantly more code on the implementation side and it does not allow for generic transformations of components. It also introduces an ambiguity, namely, what should happen if a function like `onClick` is called twice? Should both click listeners be registered or should the second click listener overwrite the first? I've seen this cause some confusion and bugs in a handful of cases, but this can usually be resolved somewhat easily. Keep this in mind when documenting your API!

## Conclusion

Your APIs should be clear and concise at the point of use and allow for flexibility and growth in the implementation. Named parameters with default values can give you this, and heterogenous maps can really help take your APIs to the next level. Again, please take these suggestions with a grain of salt. API design is far too nuanced to describe in a single blog post, and this approach should not be applied everywhere. Use it when it helps.

-----------------------------

[1] A "compatible" change is a change which does not break existing callers of an API. These stand in contrast to "breaking" changes, which require callers of an API to be updated when the API is updated.
