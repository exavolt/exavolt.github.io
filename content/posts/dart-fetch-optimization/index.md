---
title: 'Dart: optimizing simultaneous data fetching'
date: 2024-07-07T15:34:53Z
draft: false
lang: en-US
tags:
- dart
- async
- optimization
---

Let’s say we have a class `ProductService` , which is a singleton, like this:

```dart
class ProductService {
  static ProductService? _instance;
  static ProductService get instance => _instance!;
  
  Future<List<Product>> fetchProducts() async {
    // Perform the HTTP request and transform the result
  }
}
```

In various places, we can call the method `fetchProducts` as:

```dart
productService.instance.fetchProducts().then((v) { ... });
```

The problem is that this method might be called from various places simultaneously, including in multiple `FutureBuilder`s for example, which will be wasting bandwidth and unnecessarily loading the server, and potentially slowing the app down.

How can we let various places to ask for the same resource simultaneously but there will be only one fetch process?

Fortunately, one Dart Future can be subscribed by multiple listeners.

Thus, our code should be:

```dart
class ProductService {
  static ProductService? _instance;
  static ProductService get instance => _instance!;
  
  // The unifier. It's not null when the fetch process is active.
  Future<List<Product>>? _fetchProductsFuture;
  
  // This is the method that will be called from any place.
  Future<List<Product>> fetchProducts() async {
    // Conditional assignment. If it's null, we start a fetch process and
    // assign the future into it.
    _fetchProductsFuture ??= _fetchProducts()
      // When the future is complete, null the instance variable.
      .whenComplete(() => _fetchProductsFuture = null);
    // Return the instance attribute
    return _fetchProductsFuture!;
  }
  
  // The method that does the actual fetching
  Future<List<Product>> _fetchProducts() async {
    // Perform the HTTP request and transform the result
  }
}
```

Now, any simultaneous call to `productService.instance.fetchProducts()` will not create multiple HTTP request and each caller will receive the same result.

Other than for HTTP fetching, this "pattern" can also be applied to functions that do heavy computations too.
