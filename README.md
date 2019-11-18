# ImageDecoder

Image decoder in Swift using Image I/O. This implementation is based on WebKit (WebCore `ImageDecoderCG` class) and you can expect similar to how Safari handles images.

This pacakge is handy if you need to:
- Incrementally load an image;
- Decode animated images.

`ImageDecoder` supports animated images in GIF, APNG, and HEICS formats.

## Usage

If you have the complete image data you can create `ImageDecoder` and set it, `allDataReceived` indicates if the image data is complete:

```swift
let imageDecoder = ImageDecoder()
imageDecoder.setData(data, allDataReceived: true)
```

Use this approach if you read the image data from a file or downloaded from network using `URLSessionDataTask`:

```swift
let task = urlSession.dataTask(with: url) { data, _, _ in
    guard let data = data else {
        return
    }
                    
    let imageDecoder = ImageDecoder()
    imageDecoder.setData(data, allDataReceived: true)
                    
    guard let uiImage = imageDecoder.uiImage else {
        return
    }
                    
    DispatchQueue.main.async {
        self.uiImage = uiImage
    }
}
            
task.resume()
```

When you incrementally loading an image create the image data object for accumulating the image data. Pass the partial image data to `ImageDecoder` and the complete image data when loading completes. This example uses `URLSessionDelegate`:

```swift
let imageDecoder = ImageDecoder()

var imageData = Data()

func urlSession(_ session: URLSession, dataTask: URLSessionDataTask, didReceive data: Data) {
    imageData.append(data)
    imageDecoder.setData(imageData, allDataReceived: false)
}

func urlSession(_ session: URLSession, task: URLSessionTask, didCompleteWithError error: Error?) {
    imageDecoder.setData(imageData, allDataReceived: true)
}
```

## Decoding an image

`ImageDecoder` can create animated or static images. Use `createFrameImage(at index: Int, subsamplingLevel: SubsamplingLevel = .default, decodingOptions: DecodingOptions = .default) -> CGImage?`.

Creating static image:

```swift
let cgImage = imageDecoder.createFrameImage(at: 0)
```

Animated image has multiple frames:

```swift
for i in 0..<imageDecoder.frameCount {
    let cgImage = imageDecoder.createFrameImage(at: i)
    let duration = imageDecoder.frameDuration(at: i)
}
```

For convenience you can use `ImageDecoder+UIKit` extension:

```swift
extension ImageDecoder {

    /// Creates static or animated image depending on `frameCount`.
    public var uiImage: UIImage?
    
    /// Creates animated image if there is more than one frame.
    public var animatedUIImage: UIImage?
    
    /// Creates static image from the first frame.
    public var staticUIImage: UIImage?
}
```

## Displaying animated images with UIKit and SwiftUI

`UIImageView` can display animated images. In SwiftUI `Image` is static. Wrap `UIImageView` in `UIViewRepresentable` to display animated images:

```swift
struct AnimatedImage: UIViewRepresentable {
    
    let image: UIImage
    
    func updateUIView(_ uiView: UIImageView, context: UIViewRepresentableContext<AnimatedImage>) {
        uiView.image = image
    }
    

    func makeUIView(context: UIViewRepresentableContext<AnimatedImage>) -> UIImageView {
        let imageView = UIImageView(image: image)
        imageView.startAnimating()
        
        return imageView
    }
}
```

## Performance

Decoding an animated image can sometimes be slow because each frame must be decoded. Use background queue to decode animated image and make sure you use `DecodingOptions.Mode.asynchronous`. This is the default decoding mode. If you decode images on the main queue use `DecodingOptions.Mode.synchronous`.

## Misc

If you found a bug or need additional feature open an issue.

Make sure to check out [URLImage](https://github.com/dmytro-anokhin/url-image) package if you need to dowload and display an image in SwiftUI. For updates on this and other packages follow me on Twitter: [dmytroanokhin](https://twitter.com/dmytroanokhin).
