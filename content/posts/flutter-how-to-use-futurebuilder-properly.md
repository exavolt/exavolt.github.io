+++
title = 'How to use Flutter’s FutureBuilder properly'
date = 2024-06-20T02:54:27Z
draft = false
+++

When developing a Flutter app, there might be a need to display data that are not instantly available. This data needs to be prepared asynchronously to prevent freezing of the app. And while the data is being prepared, we want to display an indicator so that users know that their data is being loaded.

Fortunately, the Flutter framework has `FutureBuilder` widget that fits well for this use case.

The [official documentation](https://api.flutter.dev/flutter/widgets/FutureBuilder-class.html) does provide explanations on how to use the `FutureBuilder`, but I find that turning it into a set of practical rules makes it easier to follow.

## The problematic usage

The `FutureBuilder` has some requirements which if not satisfied it won't work correctly. The following is an example widget that utilizes a `FutureBuilder` the wrong way. While it might work in some cases, it might cause issues in other cases. Can you spot the where the part that might cause problems?

```dart
class ShowcaseView extends StatelessWidget {
  @override
  void build(BuildContext context) {
    return FutureBuilder(
      future: fetchProductList(),
      builder: (
        BuildContext context,
        AsyncSnapshot<ProductList> snapshot,
      ) {
        /* Build the content */
      }
    );
  }
}
```

The part that might cause problems is where the future is obtained, that is within the `build` method.

But why it's wrong to obtain or create the future during `build`?

In the example above, depending on the implementation of `fetchProductList`, the future might get recreated when the widget gets rebuilt. Recreating a future on every build might cause some problems:

- The widget will simply not work, i.e., the data from the completed future will never get displayed because the future keeps being recreated
- Unnecessarily wasting server's resources if the future involves fetching data from the server
- Unnecessarily drain device's battery if the future involves some heavy, on-device, computations
- The last two above can cause bad user experiences

## My rules to reliably utilize FutureBuilders

1. Make the widget a `StatefulWidget` because a `FutureBuilder` can only work in a `StatefulWidget` widget
2. Declare the future as an instance variable (a.k.a. property, attribute, member) of the state class
3. Obtain the future in the `initState` method, never during the `build`
4. Use the declared instance variable as the parameter for the `FutureBuilder` in the `build` method
    
    ```dart
    // Here's a skeleton for a widget that utilizes FutureBuilder at its
    // minimum.
    
    // 1. It must be a StatefulWidget
    class ShowcaseView extends StatefulWidget { /* ... */ }
    
    class _ShowcaseViewState extends State<ShowcaseView> {
      // 2. Declare the future as an instance variable in the state class
      Future<ProductList>? _productListFuture;
    	
      @override
      void initState() {
        // 3. Obtain or create the future in the initState
        _productListFuture = fetchProductList();
      }
    	
      @override
      Widget build(BuildContext context) {
        return FutureBuilder(
          // 4. Use the future as the parameter to FutureBuilder
          future: _productListFuture,
          builder: (
            BuildContext context,
            AsyncSnapshot<ProductList> snapshot,
          ) {
            /* Build the future-dependent content */
          }
        );
      }
    }
    ```
    
5. If the future is affected by any value passed as a parameter to the widget, ensure to override `didUpdateWidget` to re-create the future when the value changed.
    
    ```dart
    // 1.
    class ShowcaseView extends StatefulWidget {
      const ShowcaseView({
        super.key,
        required this.storeId,
      });
    
      // An example of future-affecting instance variable.
      //
      // In this example, the content of the showcase depends on the
      // store, which is specified by storeId.
      final String storeId;
    }
    
    class _ShowcaseViewState extends State<ShowcaseView> {
      // 2.
      Future<ProductList>? _productListFuture;
      
      @override
      void initState() {
        // 3. Note that the invocation has been updated to include instance variable from the   widget.
        _productListFuture = fetchProductList(storeId: widget.storeId);
      }
    
      @override
      Widget build(BuildContext context) {
        return FutureBuilder(
          // 4.
          future: _productListFuture,
          builder: (
            BuildContext context,
            AsyncSnapshot<ProductList> snapshot,
          ) {
            /* Build the future-dependent content */
          }
        );
      }
    
      // 5. Override the method didUpdateWidget to conditionally re-obtain the future
      @override
      void didUpdateWidget(covariant ShowcaseView oldWidget) {
        super.didUpdateWidget(oldWidget);
    
        // 5a. Check if the value has changed
        if (widget.storeId != oldWidget.storeId) {
          // 5b. Obtain a new future if it did change
          _productListFuture = fetchProductList(storeId: widget.storeId);
        }
      }
    }
    ```
    
6. If the state is listening to a listenable, determine whether a value change would require future recreation.

That's all. The above rules should have covered common usages of `FutureBuilder`.
