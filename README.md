# SwiftUI Reorderable

> **Note**: This is a small fork of the original [visfitness/reorderable](https://github.com/visfitness/reorderable) library, updated to be fully platform-agnostic by replacing UIKit dependencies with SwiftUI equivalents.

A pure SwiftUI structural component that allows for easy drag-and-drop reordering operations. It enables fast, `DragGesture` based interactions with its elements instead of to the "long press" based one that comes with [`.onDrag`](https://developer.apple.com/documentation/swiftui/view/ondrag(_:))/[`.draggable`](https://developer.apple.com/documentation/swiftui/view/draggable(_:)). 

This package contains a `ReorderableVStack` and a `ReorderableHStack` that work seamlessly across **iOS** and **macOS**. 

## Features

- Specify your own drag handle with the `.dragHandle()` modifier
- Disable/Enable dragging via the `.dragDisabled(_ dragDisabled: Bool)` modifier, which plays nicely with animations (as opposed to adding/removing a `.onDrag()` modifier)
- Easily customize your drag state via a `isDragged` parameter passed to your `content` view builder.
- Haptic feedback on move.

## Installation

This framework is distributed as a **Swift Package**. To use this platform-agnostic fork, add the following URL to your package list:

```
https://github.com/acyment/swift-reorderable
```

For the original library, see: [visfitness/reorderable](https://github.com/visfitness/reorderable)

To add this package to your Xcode project, follow [these instructions](https://developer.apple.com/documentation/xcode/adding-package-dependencies-to-your-app).

## Documentation

The documentation for this package can be found [here](https://visfitness.github.io/reorderable/documentation/reorderable)

## Usage

> [!NOTE]
> All the following sample use the following `struct` for their data
>
> ```swift
> private struct Sample: Identifiable {
>  var color: Color
>  var id: UUID = UUID()
>  var height: CGFloat
>  
>  init(_ color: Color, _ height: CGFloat) {
>    self.color = color
>    self.height = height
>  }
> }
> ```

### Simple Example

```swift
struct SimpleExample: View {
  @State var data = [
    Sample(.blue, 200),
    Sample(.green, 100),
    Sample(.gray, 300)
  ]
  
  var body: some View {
    ReorderableVStack($data) { $sample in
      RoundedRectangle(cornerRadius: 32, style: .continuous)
        .fill(sample.color)
        .frame(height: sample.height)
        .padding()
    }
    .padding()
  }
}
```

### Using a collection and `onMove`.

```swift
struct SimpleExample: View {
  @State var data = [
    Sample(.blue, 200),
    Sample(.green, 100),
    Sample(.gray, 300)
  ]
  
  var body: some View {
    ReorderableVStack(data, onMove: { from, to in
      withAnimation {
        data.move(fromOffsets: IndexSet(integer: from),
                  toOffset: (to > from) ? to + 1 : to)
      }
    }) { sample in
      RoundedRectangle(cornerRadius: 32, style: .continuous)
        .fill(sample.color)
        .frame(height: sample.height)
        .padding()
    }
    .padding()
  }
}
```

### With Custom Drag Handle and Dragging Effect

```swift
struct SimpleExample: View {
  @State var data = [
    Sample(.blue, 200),
    Sample(.green, 100),
    Sample(.gray, 300)
  ]
  
  var body: some View {
    ReorderableVStack($data) { $sample, isDragged in // <------ Notice the additional `isDragged` parameter
      ZStack(alignment: .leading) {
        RoundedRectangle(cornerRadius: 32, style: .continuous)
          .fill(sample.color)
          .frame(height: sample.height)
        
        Image(systemName: "line.3.horizontal")
          .foregroundStyle(.secondary)
          .padding()
          .offset(x: 16)
          // This will now be the only place users can drag the view from
          .dragHandle() // <------------
      }
      .scaleEffect(isDragged ? 1.1: 1)
      .animation(.easeOut, value: isDragged)
      .padding()
    }.padding()
  }
}
```

### When Part of a `ScrollView`

> [!WARNING]
> Because this package doesn't rely on SwiftUI's native `onDrag`, it also doesn't automatically trigger auto-scrolling when users drag the element to the edge of the parent/ancestor `ScrollView`. To enable this behavior, the `autoScrollOnEdges()` modifier needs to be applied to the `ScrollView`.

```swift
struct SimpleExample: View {
  @State var data = [
    Sample(.blue, 200),
    Sample(.green, 200),
    Sample(.gray, 300),
    Sample(.mint, 200),
    Sample(.purple, 300),
    Sample(.orange, 200)
  ]
  
  var body: some View {  
    ScrollView {
      ReorderableVStack($data) { $sample in
        RoundedRectangle(cornerRadius: 32, style: .continuous)
          .fill(sample.color)
          .frame(height: sample.height)
          .padding()
      }.padding()
    }.autoScrollOnEdges() // <------- This modifier enables the autoscrolling
  }
}
```

### Nested `ReorderableHStack` in `ReorderableVStack`

```swift
private struct Sample2D: Identifiable {
  var id: UUID = UUID()
  var row: [Sample]
}

struct SimpleExample: View {
  @State var data: [Sample2D] = [
    .init(row: [.init(.blue, 200), .init(.green, 100), .init(.gray, 200)]),
    .init(row: [.init(.red, 200), .init(.mint, 100), .init(.purple, 200)]),
    .init(row: [.init(.indigo, 200), .init(.teal, 100), .init(.yellow, 200)]),
  ]

  var body: some View {
    ReorderableVStack($data) { $sample in
      HStack {
        ZStack {
          RoundedRectangle(cornerRadius: 24, style: .continuous)
            .fill(Color.orange)
            .frame(width: 64, height: 64)
            .padding()
         
          Image(systemName: "line.3.horizontal")
            .foregroundStyle(.secondary)
            .padding()
        }
        .dragHandle()
        
        ReorderableHStack($sample.row) { $sample in
          RoundedRectangle(cornerRadius: 24, style: .continuous)
            .fill(sample.color)
            .frame(width: 64, height: 64)
            .padding()
        }
      }
    }
  }
}
```


## Copyright and License

Copyright [Vis Fitness Inc](https://vis.fitness). Licensed under the [MIT License](https://github.com/visfitness/reorderable/blob/main/LICENSE)
