# SourceryTemplates

This repository contains some templates (in the `Templates/` directory) to use for Code Generation in Swift with [Sourcery](http://github.com/krzysztofzablocki/Sourcery).

You can see them in action by opening the `TemplatesDemo.xcodeproj` Xcode project: its `UnitTests` target contains some use case examples for each template, and some code using the generated code in each associated `XCTestCase`.

## Type Erasure

This template provides type erasure to any protocol you annotated accordingly.

### Usage

* Annotate the protocol to type-erase using `// sourcery: TypeErase = X` where `X` is the `associatedtype` to erase
* Use the template in [`Templates/TypeErase.stencil`](https://github.com/AliSoftware/SourceryTemplates/blob/master/Templates/TypeErase.stencil) with [Sourcery](http://github.com/krzysztofzablocki/Sourcery) to generate Type-Erased code for this annotated protocol(s)
* Add the generated `TypeErase.generated.swift` file to your Xcode project
* Profit!

### Examples

You can find some examples in the [`UnitTests/TypeErasure`](https://github.com/AliSoftware/SourceryTemplates/tree/master/UnitTests/TypeErasure) directory in this repo, like those:

```swift
// sourcery: TypeErase = PokemonType
protocol Pokemon {
  associatedtype PokemonType
  func attack(move: PokemonType)
}
```

```swift
// sourcery: TypeErase = Model
protocol Row {
  associatedtype Model

  var sizeLabelText: String { get set }

  func configure(model: Model)
}
```

You can look at the [file generated by Sourcery](https://github.com/AliSoftware/SourceryTemplates/blob/master/TypeErasure/UnitTests/Generated/TypeErase.generated.swift) from those examples using the provided [`Templates/TypeErase.stencil`](https://github.com/AliSoftware/SourceryTemplates/blob/master/Templates/TypeErase.stencil) template. Magic! 🎩✨

### More Info

This is a first draft of the template, so please don't hesitate to improve it.

I haven't tested a lot of real-world use cases yet so some corner cases are probably missing but feel free to submit a PR to fix them!

The template generates the same code structure as the one explained in [this BigNerdRanch article](https://www.bignerdranch.com/blog/breaking-down-type-erasures-in-swift/) that inspired me to come up with a working template.

You can also find more information about Type-Erasure by watching [Gwendolyn's talk at Try! Swift](https://news.realm.io/news/tryswift-gwendolyn-weston-type-erasure/) — from which I borrowed the Pokemon examples to test that template.

## AutoInterface

This template allows to auto-generate a protocol matching the API of a `class`/`struct`.

### Usage

* Create an empty phantom protocol `protocol AutoInterface {}` somewhere in your code.
* Make your classes or structs conform to `AutoInterface` to opt-in
* Optionally, you can customize the name of the generated `protocol` for each type using annotations:
  * By default, the generated protocol as the same name as the type it was generated from, but prefixed with an `I` (e.g. `protocol IWebService` for `class WebService: AutoInterface`
  * You can use `// sourcery: AutoInterfacePrefix = …` to change that prefix
  * You can use `// sourcery: AutoInterfaceSuffix = …` to add a suffix
  * You can use `// sourcery: AutoInterface = …` to force an exact name (in that case the `AutoInterfacePrefix` & `AutoInterfaceSuffix` are ignored if present)
* Use the template in [`Templates/AutoInterface.stencil`](https://github.com/AliSoftware/SourceryTemplates/blob/master/Templates/AutoInterface.stencil) with [Sourcery](http://github.com/krzysztofzablocki/Sourcery) to generate a `protocol` for each opted-in type and automatically make your type conform to that new protocol
* Add the generated `AutoInterface.generated.swift` file to your Xcode project
* Profit!

### Examples

You can find some examples in the [`UnitTests/AutoInterface`](https://github.com/AliSoftware/SourceryTemplates/tree/master/UnitTests/AutoInterface) directory in this repo, like this:

```swift
class UserWSClient: AutoInterface {
  struct User {
    let id: Int
    let name: String
  }

  let baseURL: URL = URL(string: "https://example.com/api")!
  
  func fetchUsers() -> [User] {
    return (0..<10).map { User(id: $0, name: "User\($0)") }
  }

  func fetchUser(id: Int) -> User? {
    return User(id: id, name: "User\(id)")
  }
}
```

Will generate a `protocol IUserWSClient` containing all the functions and properties of your type and make your type conform to that protocol:

```swift
protocol IUserWSClient {
	var baseURL: URL { get }
	func fetchUsers() -> [WSClient.UserWSClient.User]
	func fetchUser(id: Int) -> WSClient.UserWSClient.User?
}
extension UserWSClient: IUserWSClient {}
```

You can look at the [file generated by Sourcery](https://github.com/AliSoftware/SourceryTemplates/blob/master/UnitTests/Generated/AutoInterface.generated.swift) from this example using the provided [`Templates/AutoInterface.stencil`](https://github.com/AliSoftware/SourceryTemplates/blob/master/Templates/AutoInterface.stencil) template. Magic! 🎩✨

## AutoPropertiesProtocol

This template allows you to generate one protocol for each property of your type, each protocol exposing that single property.

This allows you to later leverage protocol composition to express exactly which properties of that type you're gonna use and want to be able to access.

This pattern is typically useful to [leverage protocol composition for Dependency Injection](http://merowing.info/2017/04/using-protocol-compositon-for-dependency-injection/), which mixes the benefits of having a single struct containing all your dependencies (instead of passing them all one by one in your constructor) and the benefits of compile-type safety + explicit list of dependencies you want without exposing too much

### Usage

* Declare a phantom protocol `protocol AutoPropertiesProtocol {}` somewhere in your code
* Make the class you want to opt-it for this feature to conform to this phantom protocol
* Optionally, you can annotate your properties with the `PropertiesProtocolPrefix` and/or `PropertiesProtocolSuffix` annotations to provide a custom prefix/suffix for the name of the generated protocols
  * By default (if the annotation is not set / has no value), the prefix will be `Has` and there will be no suffix
  * Reminder: if you want _all_ the properties of your type to have the same prefix/suffix, you can use `// sourcery:begin: PropertiesProtocolPrefix = …` at the beginning of your type + `// sourcery:end` at the end to apply the same annotation(s) to all the variables inside that scope.
* Alternatively, you can also optionally annotate your properties with the `PropertiesProtocol` annotation to give an exact name for the protocol to generate for that property.
* Finally, use the template in [`Templates/AutoPropertiesProtocol.stencil`](https://github.com/AliSoftware/SourceryTemplates/blob/master/Templates/AutoPropertiesProtocol.stencil) with [Sourcery](http://github.com/krzysztofzablocki/Sourcery) to generate all the necessary protocols

### Examples

You can find some examples in the [`UnitTests/AutoPropertiesProtocol`](https://github.com/AliSoftware/SourceryTemplates/tree/master/UnitTests/AutoPropertiesProtocol) directory in this repo, like this one:

```swift
// sourcery:begin: PropertiesProtocolPrefix = I, PropertiesProtocolSuffix = Container
class Dependencies: AutoPropertiesProtocol {
  let webServiceClient: WebServiceClient
  let loginManager: LoginManager
  let cartManager: CartManager

  init(wsClient: WebServiceClient, loginManager: LoginManager, cartManager: CartManager) {
    self.webServiceClient = wsClient
    self.loginManager = loginManager
    self.cartManager = cartManager
  }
}
// sourcery:end
```

With this example above, you can then use `Dependencies` as a single `struct` containing _all_ the dependencies you need to pass throughout your app workflow, but still keep type safety and explicitness by saying exactly which dependencies of that container you allow to use:

```swift
class LoginScreen {
  typealias Dependencies = IWebServiceClientContainer & ILoginManagerContainer
  private let deps: Dependencies
  init(deps: Dependencies) {
    self.deps = deps
  }

  func login() {
    print(self.deps.webServiceClient)
    print(self.deps.loginManager)
    // print(self.deps.cartManager) // Can't access this, which is a good thing :)
  }
}
```

You can look at the [file generated by Sourcery](https://github.com/AliSoftware/SourceryTemplates/blob/master/AutoPropertiesProtocol/UnitTests/Generated/AutoPropertiesProtocol.generated.swift) from those examples using the provided [`Templates/AutoPropertiesProtocol.stencil`](https://github.com/AliSoftware/SourceryTemplates/blob/master/Templates/AutoPropertiesProtocol.stencil) template. Magic! 🎩✨

## Other templates

* [Sourcery itself has some template examples](https://github.com/krzysztofzablocki/Sourcery/tree/master/Templates)
* You can also find a template by **@Liquidsoul** [on his own repo](https://github.com/Liquidsoul/Sourcery-AutoJSONSerializable) to auto-generate code for JSON serialization & deserialization.
