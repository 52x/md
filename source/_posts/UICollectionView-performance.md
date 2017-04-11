title: UICollectionView性能优化
date: 2015-12-09 15:57:26
categories: iOS
tags: [swift]
---
![瀑布流展示图片](/images/UICollectionView/532index.png)
在开发532Movie的过程中，遇到了UICollectionView卡顿问题。本文提供了几种优化的思路，力求让UICollectionView丝丝润滑。

<!-- more -->

大部分人用UICollectionView展示图片时都会遇到性能问题，尤其在滚动的时候，会卡顿明显。可以从如下价格方面入手：

# 异步加载
斯坦福CS193P课程中，在讲UITableVIew的时候，再次强调了所有网络操作都要异步，不能阻塞UI进程。可以参考苹果官方的LazyTableImages Demo，虽然是UITableView的Demo，但是原理完全相同。

# Reuse Cell
当有上千上万个cell时，iOS并不会真的去创建那么多cell，创建cell的代价太高。系统只会创建屏幕可视的那几个cell，当cell滑出屏幕时，会将cell放入内存池，以供新出现的cell复用。


我在项目中使用Alamofire异步请求图片资源，但是在滚动的时候依旧有一点点跳跃地卡顿感。继续优化

# NSCache

参考[Alamofire Tutorial Part 2: Progress and Caching](http://www.raywenderlich.com/87595/intermediate-alamofire-tutorial)这篇文章中有如下一段话:
`When you scroll quickly through the photo browser, you’ll notice that you can send cells off the screen whose image requests are still active. In fact, the image request still runs to completion, but the downloaded photo and associated data is just discarded.
Additionally, when you return to earlier cells you have to make a network request again for the photo — even though you just downloaded it a moment ago. You can definitely improve on this bandwidth-wasting design!
You’ll do this by caching retrieved images so they don’t have to be retrieved numerous times; as well, you’ll cancel any in-progress network requests if the associated cell is dequeued before the request completes.`
从两个方面阐述了为何要使用NSCache缓存图片，首先当你滑动速度较快，图片还没有加载结束但是cell已经滚出屏幕，此时我们可以将这个网络请求取消；其次如果滑回之前的cell，由于没有缓存，会再次发起网络请求，我们要避免对同一个资源请求多次。
具体代码如下:
```swift
let imageCache = NSCache()
override func collectionView(collectionView: UICollectionView, cellForItemAtIndexPath indexPath: NSIndexPath) -> UICollectionViewCell {
  let cell = collectionView.dequeueReusableCellWithReuseIdentifier(PhotoBrowserCellIdentifier, forIndexPath: indexPath) as! PhotoBrowserCollectionViewCell
 
  let imageURL = (photos.objectAtIndex(indexPath.row) as! PhotoInfo).url
 
  // 1
  cell.request?.cancel()
 
  // 2
  if let image = self.imageCache.objectForKey(imageURL) as? UIImage {
    cell.imageView.image = image
  } else {
    // 3
    cell.imageView.image = nil
 
    // 4
    cell.request = Alamofire.request(.GET, imageURL).validate(contentType: ["image/*"]).responseImage() {
      (request, _, image, error) in
      if error == nil && image != nil {
        // 5
        self.imageCache.setObject(image!, forKey: request.URLString)
 
        // 6
        cell.imageView.image = image
      } else {
        /*
        If the cell went off-screen before the image was downloaded, we cancel it and
        an NSURLErrorDomain (-999: cancelled) is returned. This is a normal behavior.
        */
      }
    }
  }
 
  return cell
}
```
# Instrument分析+Kingfisher
在实现了第三步之后，iPhone6上面几乎没有卡顿感，但是我的iPad 3真机由于屏幕图片数目过多，依然会有卡顿。
使用Instrument分析时间开销，Cell中没有复杂的UI，也没有使用UIFontDescriptor这类耗时的方法，每个Cell高度都是固定的。最后分析的结果仍是UIImage的加载最耗时，调研了一番swift图片库，决定使用Kingfisher试试。换上Kingfisher加载图片后，瞬间腰不酸背不痛，UICollectionView变得异常流畅。
Kingfisher相比于Alamofire，在请求、载入图片时做了更多的优化，这个库支持取消进程、多级缓存、缓存管理、后台压缩图片渲染等技术。
当用瀑布流的形式展示图片时，不妨考虑使用Kingfisher优化性能。