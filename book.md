# 必要的图像优化



**我们应该对图像进行自动化压缩。**

应该自动优化图像。当最佳方案改变时，这点很容易被以往，尤其是在未进行**流水线式构建**时这一步很容易丢失。进行自动化：在你的构建过程中使用[imagemin](https://github.com/imagemin/imagemin) 或 [libvips](https://github.com/jcupitt/libvips) 工具，也存在着更多的替代方案。

大多数CDN(例如：[Akamai](https://www.akamai.com/us/en/solutions/why-akamai/image-management.jsp)) 和第三方解决方案，例如 [Cloudinary](https://cloudinary.com), [imgix](https://imgix.com), [Fastly’s Image Optimizer](https://www.fastly.com/io/), [Instart Logic’s SmartVision](https://www.instartlogic.com/technology/machine-learning/smartvision)或[ImageOptim API](https://imageoptim.com/api) 都提供有全面的图像自动优化方法。

你在阅读博客并调整配置文件上花的时间要大于使用第三方包月服务（Cloudinary有免费套餐）。如果你因为成本或延迟问题而不想将这项工作外包出去，那么上面提到的这些开源方案也是可靠的。像 [Imageflow](https://github.com/imazen/imageflow) 或 [Thumbor](https://github.com/thumbor/thumbor) 提供可选的自动托管方案。

**每个人应该对他们的图像进行有效的压缩。**

至少要使用 [ImageOptim](https://imageoptim.com/)。它可以在保持显示效果的同时能显著的降低文件大小。 在Windows和Linux平台也有各自可用的 [版本](https://imageoptim.com/versions.html) 。

更具体点来说，通过 [MozJPEG](https://github.com/mozilla/mozjpeg)（q = 80或更低，适用于Web内容）运行您的JPEG格式图像并考虑支持使用渐进式JPEG编码方式。PNG格式使用 [pngquant](https://pngquant.org/) ，SVG格式使用 [SVGO](https://github.com/svg/svgo)。要显式的删除图像的元数据（使用pngquant时可使用`--strip` ）来避免图像大小发生膨胀。使用[H.264](https://en.wikipedia.org/wiki/H.264/MPEG-4_AVC) 编码压缩方式的视频文件（或Chrome、Firefox和Opera支持的[WebM](https://www.webmproject.org/)）来替代体积过大的GIF动图，如果不行也至少使用 [Giflossy](https://github.com/kornelski/giflossy)进行处理。如果你有额外的CPU资源，并且需要高于网络平均质量的图像，还能接受较慢的编码速度，可以尝试使用[Guetzli](https://research.googleblog.com/2017/03/announcing-guetzli-new-open-source-jpeg.html)。

一些浏览器通过HTTP的Accept请求标头来声明所支持的图像格式。这可以用来有条件的提供一些格式，例如：基于Blink的浏览器（如Chrome）的有损的 [WebP](https://developers.google.com/speed/webp/)格式或其他浏览器可备用的JPEG / PNG等格式。

你可以做的有更多。有用于生成和提供`srcset`断点的工具。也可以在基于Blink的浏览器中使用 [client-hints](https://developers.google.com/web/updates/2015/09/automating-resource-selection-with-client-hints)（客户端提示）的方式自动选择资源，并且可以通过指定[Save-Data](https://developers.google.com/web/updates/2016/02/save-data)（数据持久化）标头提示浏览器选择“data savings（数据缓存）”，这样就可以向的用户发送更少字节的内容。

（译者注：关于Save-Data的介绍可参考这篇文章[Help Your Users 'Save-Data'](https://css-tricks.com/help-users-save-data/)）

你生成的图像文件大小越小，对用户提供的网络使用体验就越好，尤其是使用移动设备的用户。在这篇文章中，我们将探讨通过现代压缩技术减少图像尺寸的方法，同时将对图像质量的影响降至最低。



<details>
<summary><h2>目录</h2></summary>
<p>
<ul>
        <li><a href="#introduction">Introduction</a></li>
        <li><a href="#do-my-images-need-optimization">How can I tell if my images need to be optimized?</a></li>
        <li><a href="#choosing-an-image-format">How do I choose an image format?</a></li>
        <li><a href="#the-humble-jpeg">The humble JPEG</a></li>
        <li><a href="#jpeg-compression-modes">JPEG compression modes</a>
                <ul>
                        <li><a href="#the-advantages-of-progressive-jpegs">The advantages of Progressive JPEGs</a></li>
                        <li><a href="#whos-using-progressive-jpegs-in-production">Who’s using Progressive JPEGs in production?</a></li>
                        <li><a href="#the-disadvantages-of-progressive-jpegs">The disadvantages of Progressive JPEGs</a></li>
                        <li><a href="#how-to-create-progressive-jpegs">How do you create Progressive JPEGs?</a></li>
                        <li><a href="#chroma-subsampling">Chroma (or Color) Subsampling</a></li>
                        <li><a href="#how-far-have-we-come-from-the-jpeg">How far have we come from the JPEG?</a></li>
                        <li><a href="#optimizing-jpeg-encoders">Optimizing JPEG Encoders</a></li>
                        <li><a href="#what-is-mozjpeg">What is MozJPEG?</a></li>
                        <li><a href="#what-is-guetzli">What is Guetzli?</a></li>
                        <li><a href="#mozjpeg-vs-guetzli">How does MozJPEG compare to Guetzli?</a></li>
                        <li><a href="#butteraugli">Butteraugli</a></li>
                </ul>
        </li>
        <li><a href="#what-is-webp">What is WebP?</a>
                <ul>
                        <li><a href="#how-does-webp-perform">How does WebP perform?</a></li>
                        <li><a href="#whos-using-webp-in-production">Who’s using WebP in production?</a></li>
                        <li><a href="#how-does-webp-encoding-work">How does WebP encoding work?</a></li>
                        <li><a href="#webp-browser-support">WebP browser support</a></li>
                        <li><a href="#how-do-i-convert-to-webp">How do I convert my images to WebP?</a></li>
                        <li><a href="#how-do-i-view-webp-on-my-os">How do I view WebP images on my OS?</a></li>
                        <li><a href="#how-do-i-serve-webp">How do I serve WebP?</a></li>
                </ul>
        </li>
        <li><a href="#svg-optimization">SVG optimization</a></li>
        <li><a href="#avoid-recompressing-images-lossy-codecs">Avoid recompressing images with lossy codecs</a></li>
        <li><a href="#reduce-unnecessary-image-decode-costs">Reduce unnecessary image decode and resize costs</a>
                <ul>
                        <li><a href="#delivering-hidpi-with-srcset">Delivering HiDPI images using <code>srcset</code></a></li>
                        <li><a href="#art-direction">Art direction</a></li>
                </ul>
        </li>
        <li><a href="#color-management">Color management</a></li>
        <li><a href="#image-sprites">Image spriting</a></li>
        <li><a href="#lazy-load-non-critical-images">Lazy-load non-critical images</a></li>
        <li><a href="#display-none-trap">Avoiding the <code>display: none;</code> trap</a></li>
        <li><a href="#image-processing-cdns">Does an image processing CDN make sense for you?</a></li>
        <li><a href="#caching-image-assets">Caching image assets</a></li>
        <li><a href="#preload-critical-image-assets">Preloading critical image assets</a></li>
        <li><a href="#performance-budgets">Performance Budgets For Images</a></li>
        <li><a href="#closing-recommendations">Closing recommendations</a></li>
        <li><a href="#trivia">Trivia</a></li>
</ul>
</p>
</details>



## <a id="introduction" href="#introduction">简介</a>

**图像仍然是现如今网络空间膨胀额主要原因**

图像占用着大量的互联网带宽，因为通常来说图像的文件大小都比较大。根据[HTTP Archive](http://httparchive.org/)的数据，互联网传输过程中60%的数据都是由JPEG、PNG和GIF等图像组成。截止到2017年，平均大小为3.00MB的网站内容中，图像文件约占到 [1.7MB](http://httparchive.org/interesting.php#bytesperpage)。

据[Tammy Everts](https://www.linkedin.com/in/tammyeverts)所言，向页面添加图像或使现有图像变大已被证明可以提高[转换率](https://calendar.perfplanet.com/2014/images-are-king-an-image-optimization-checklist-for-everyone-in-your-organization/)（译者注：此处的转换率指的是网站的访问者转换为活跃用户）。图像不太可能消失，因此在有效的最小化图像压缩策略上的投资就变得非常重要。

<img
        class="lazyload small"
        data-src="images/book-images/Modern-Image00-large.jpg"
        alt="Fewer images per page create more conversions. 19 images per page on average converted better than 31 images per page on average." />





> Per from 2016, images were the 2nd highest predictor of conversions with the best pages having 38% fewer images.根据2016年 [Soasta/Google 的研究](https://www.thinkwithgoogle.com/marketing-resources/experience-design/mobile-page-speed-load-time/)，图像是转换率的第二高预测因素，最佳页面的图像减少了38％。



图像优化包含多种方式来减小图像大小，最终取决于你想要达到什么程度的视觉效果。

<img
        class="lazyload small"
        data-src="images/book-images/image-optimisation-large.jpeg"
        alt="Image optimization covers a number of different techniques" />

> **图像优化：** 选择正确的格式，合理的压缩，相较于可以懒加载的图像优先加载关键图像。



常见的图像优化包括：压缩、使用 [`<picture>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/picture)或[`<img srcset>`](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images)来根据屏幕大小响应地降低尺寸，并且降低图像大小能降低解码成本。






<img
        class="lazyload small"
        data-src="images/book-images/chart_naedwl-large.jpg"
        alt="A histogram of potential image savings from the HTTP Archive validating the 30KB of potential image savings at the 95th percentile." />





> 根据[HTTP Archive](http://jsfiddle.net/rviscomi/rzneberp/embedded/result/)表示，95%图像预加载就只有30KB！（可查看累积分布函数）



对我们来说，有足够的空间来对图像进行批量优化。



<img
        class="lazyload small"
        data-src="images/book-images/image-optim-large.jpg"
        alt="ImageOptim in use on Mac with a number of images that have been compressed with savings over 50%" />

> ImageOptim是免费的，通过现代压缩技术和剔除不必要的EXIF元数据来减少图像大小。



如果你是一个设计师，你也可以使用ImageOptim为Sketch制作的[插件](https://github.com/ImageOptim/Sketch-plugin)，能在导出的时候能对你的资产进行优化。我发现它能节省好多时间。





## <a id="do-my-images-need-optimization" href="#do-my-images-need-optimization">如何判断图像是否需要优化？</a>

通过[WebPageTest.org](https://www.webpagetest.org/)站点进行审计，它会高亮显示可优化图像的机会。（详见“图像压缩”部分）

<figure>
<picture>
<source
        data-srcset="images/book-images/Modern-Image1-small.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="images/book-images/Modern-Image1-medium.jpg"
        media="(max-width: 1024px)" />



<img
        class="lazyload small"
        data-src="images/book-images/Modern-Image1-large.jpg"
        alt="WebPage test supports auditing for image compression via the compress images section" />





> WebPageTest报告的“压缩图像”部分列出了可以更有效地压缩的图像的方法以及预估可节省的文件大小。



<img
        class="lazyload small"
        data-src="images/book-images/Modern-Image2-medium.jpg"
        alt="image compression recommendations from webpagetest" />



[Lighthouse](https://developers.google.com/web/tools/lighthouse/) 为最佳性能表现提供审计。它包含审核图像优化方式，并可以为进一步压缩图像方式提供建议，或指出屏幕未显示的图像可以使用延迟加载方式进行加载。

从Chrome 60开始，Lighthouse在开发者工具的 [Audits面板](https://developers.google.com/web/updates/2017/05/devtools-release-notes#lighthouse) 中提供使用。



<img
        class="lazyload small"
        data-src="images/book-images/hbo-large.jpg"
        alt="Lighthouse audit for HBO.com, displaying image optimisation recommendations" />



> Lighthouse可以审核Web的性能、最佳实践和渐进式Web应用程序功能。（注：Performance、Best Practices、Progressive Web App为audits的复选项。Best Practices表示Web是否按照最佳方法进行现代Web开发）



您也可能还熟悉其他性能审核工具，如 [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/) 或 [Website Speed Test](https://webspeedtest.cloudinary.com/)，其中包括详细的图像分析审核。



## <a id="choosing-an-image-format" href="#choosing-an-image-format">如何选择图像格式?</a>

正如Ilya Grigorik在他的杰作 [图像优化指南](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/image-optimization)中所指出的，“合适的”图像格式是视觉效果和功能要求的均衡（注：combination相比组合翻译成均衡更通顺些）。 你在使用位图还是矢量图形呢？

<img
        class="lazyload very-small"
        data-src="images/book-images/rastervvector-small.png"
        alt="vector vs raster images"
         />

[位图](https://en.wikipedia.org/wiki/Raster_graphics)通过对像素矩阵内的每一个像素的值进行编码来表示图像，与分辨率或缩放无关。WebP或者应用广泛的格式（例如PNG，JPEG）都可以很好的在保持图像内容的情况下处理这些图像。我们讨论过的Guetzli，MozJPEG或其他格式也同样适用于位图。

[矢量图形](https://en.wikipedia.org/wiki/Vector_graphics) 使用点、线和多边形来表示图像和格式，使用简单的几何形状（例如logo）提供高分辨率和缩放，像SVG在处理这种情况会更好。（TODO）

[矢量图形](https://en.wikipedia.org/wiki/Vector_graphics) use points, lines and polygons to represent images and formats using simple geometric shapes (e.g. logos) offering a high-resolution and zoom like SVG handle this use case better.

错误的格式会让你付出代价，选择使用正确格式的逻辑流程可能充满风险，因此使用其他格式保存图像进行实验可以规避这些风险。（TODO）

Jeremy Wagner 在他的图像优化演讲中谈到了评估使用格式时值得考虑的权重（[示例](http://jlwagner.net/talks/these-images/#/2/2) ）。

## <a id="the-humble-jpeg" href="#the-humble-jpeg">精简的JPEG</a>

JPEG应该是世界上使用最广泛的图像格式。 如前文所述，HTTP Archive在抓取网站上的图像的 [统计结果](http://httparchive.org/interesting.php) 可以看到其中有45％是JPEG。你的手机，数码单反相机，老式网络摄像头等这些设备都支持这种编解码方式。 这种格式也很久远，可以追溯到1992年时首次发布。 在那段时间里，它被已经被进行了大量的研究来尝试改进它的编码方式。

JPEG是一种有损压缩算法，它丢弃部分信息来节省存储空间，并且在尝试保持视觉效果的基础上尽可能使文件大小尽可能的小。

**您的项目可接受什么程度图像质量？**

JPEG等格式最适合具有多个颜色区域的照片或图像。 大多数优化工具允许你设置期望的压缩级别; 较高的压缩级别会减小文件大小，但会产生引入伪像，光晕等额外噪声。（译者注：[artifacts](https://en.wikipedia.org/wiki/Compression_artifact#Artistic_use) 指的是过度锐化或细节丢失等影响视觉显示效果的瑕疵，由于JPEG的DTC压缩导致出现方块现象，可以参考论坛的[讨论](http://forum.xitek.com/forum.php?mod=viewthread&tid=1605692&page=1&ordertype=1) 。后续该词翻译为伪像)。

<img
        class="lazyload"
        data-src="images/book-images/Modern-Image5-large.jpg"
        alt="JPEG compression artifacts can be increasingly perceived as we shift from best quality to lowest" />

> JPEG：可感知的JPEG压缩伪影会随着我们从最佳质量到最低质量的转变而显著增加。请注意，一个工具中的图像质量分数可能与另一个工具中的质量分数有所不同。

当选择需要设置图像的质量时，请考虑您的图像属于哪个范畴：

- **最佳质量**–当质量比带宽更重要。这可能是因为图像在您的设计中具有很高的突出度或以全分辨率显示。
- **良好的质量**-当您更关心传输较小的文件，但不想对图像质量造成太大的影响。用户相对更关心图像质量。
- **低质量**-当你更关心带宽，适量的图像降级是可以接受的。这些图像适用于不稳定或较差的网络条件。
- **最低质量**-节省带宽更为重要。为了更快的加载页面，用户能接受相对较差的用户体验。

接下来，让我们将讨论一下JPEG的压缩模式，因为这些模式对体验性能变化有很大的影响。

<aside class="note"><b>Note:</b> 有时我们可能高估了用户对图像质量的需求。图像质量可以被视为与未压缩前理想的图像的偏差，这种偏差具有主观性。（100%图像质量可被理解为无偏差）</aside>
## <a id="jpeg-compression-modes" href="#jpeg-compression-modes">JPEG 压缩模式</a>

JPEG图像格式具有许多不同的[压缩模式](http://cs.haifa.ac.il/~nimrod/Compression/JPEG/J5mods2007.pdf)。 三种流行的模式是基线方式（顺序），渐进式（PJPEG）和无损方式。

**基线（或顺序）JPEG和渐进式JPEG有何不同？**

基线JPEG（大多数图像编辑和优化工具的默认值）以相对简单的方式进行编码和解码：从上到下。在缓慢或不稳定的连接上以基线方式加载JPEG时，用户会先看到图像的顶部，随着图像的加载会逐行显示更多的内容。无损JPEG类似，但压缩比较小。

<img
        class="lazyload"
        data-src="images/book-images/Modern-Image6-large.jpg"
        alt="baseline JPEGs load top to bottom" />





> 基线JPEG从上到下加载，而渐进式JPEG从模糊加载到清晰。



渐进式jpeg将图像分成若干个扫描阶段。第一次扫描显示位于模糊或低质量的图像区域，随后的扫描再提高图像质量。把这看作是“逐步”加载。图像的每次“扫描”都会增加细节级别。合并后，将创建显示完整质量的图像。

<img
        class="lazyload"
        data-src="images/book-images/Modern-Image7-large.jpg"
        alt="progressive JPEGs load from low-resolution to high-resolution" />

> 基线JPEG从上到下加载图像。渐进式JPEG从低分辨率（模糊）加载到高分辨率。 Pat Meenan编写了一个[交互式工具](http://www.patrickmeenan.com/progressive/view.php?img=https%3A%2F%2Fwww.nps.gov%2Fplanyourvisit%2Fimages%2FGrandCanyonSunset_960w.jpg) 来测试和了解渐进式JPEG加载方式。

通过删除数码相机或编辑器添加的[EXIF数据](http://www.verexif.com/en/)，优化图像的[Huffman表](https://en.wikipedia.org/wiki/Huffman_coding)或重新扫描图像，可以实现无损的JPEG优化。 像[jpegtran](http://jpegclub.org/jpegtran/) 这样的工具通过重新排列压缩数据而不会降低图像质量来实现图像的无损压缩。 [jpegrescan](https://github.com/kud/jpegrescan), [jpegoptim](https://github.com/tjko/jpegoptim) 和 [mozjpeg](https://github.com/mozilla/mozjpeg)（我们将在稍后介绍）也支持无损JPEG压缩。

### <a id="the-advantages-of-progressive-jpegs" href="#the-advantages-of-progressive-jpegs">渐进式JPEG的优点</a>

渐进式JPEG在加载时提供图像的低分辨率“预览”的方式，与自适应图像相比提高了用户的体验，用户可以感觉图像加载速度更快。

在较慢的3G连接上，这种方式允许用户在仅接收到部分文件时（粗略地）查看图像中的内容，并决定是否等待其完全加载。 这比基线JPEG提供的图像从上到下显示方式更令人愉快。

<img
        class="lazyload small"
        data-src="images/book-images/pjpeg-graph-large.png"
        alt="impact to wait time of switching to progressive jpeg" />

> 2015年，Facebook转向了使用渐进式JPEG（[用于iOS应用程序](https://code.facebook.com/posts/857662304298232/faster-photos-in-facebook-for-ios/)），数据使用量减少了10%。他们能够以比以前快15%的速度显示高质量的图像，优化图像的加载时间，如上图所示。



对于超过10KB的图像，渐进式JPEG可以改善压缩能力，与基线/简单JPEG相比，带宽减少 [2-10%](http://www.bookofspeed.com/chapter5.html) 。 它们的压缩比更高，这要归功于JPEG中的每次扫描都能够使用自己专用的可选[Huffman 表](https://en.wikipedia.org/wiki/Huffman_coding)。 现代JPEG编码器（例如：[libjpeg-turbo](http://libjpeg-turbo.virtualgl.org/)，MozJPEG等）利用PJPEG的灵活性来更好地打包数据。

**Note：**为什么渐进式JPEG压缩得更好？ 基线JPEG的数据库块一次编码一个。 利用渐进式JPEG，可以将多个数据块的类似[离散余弦变换](https://en.wikipedia.org/wiki/Discrete_cosine_transform) 系数编码在一起，从而实现更好的压缩。

渐进式JPEG的另一个优点是在HTTP2上，页面和第一个扫描层同时加载，这[提高了用户查看初始图像内容的速度](https://calendar.perfplanet.com/2016/even-faster-images-using-http2-and-progressive-jpegs/) ，并使浏览器能够更快地布局页面元素。 将其与渐进式JPEG的定制扫描层相结合，例如：通过[向mozjpeg提供自定义扫描文件](https://calendar.perfplanet.com/wp-content/uploads/2016/12/scans.txt) 或使用[Cloudinary的自定义PJPEG选项](http://cloudinary.com/blog/progressive_jpegs_and_green_martians) ，可以更快地为用户呈现真正有意义的图像内容。



### <a id="whos-using-progressive-jpegs-in-production" href="#whos-using-progressive-jpegs-in-production">谁在产品中使用渐进式JPEG？</a>

-  [Twitter.com使用渐进式JPEG](https://www.webpagetest.org/performance_optimization.php?test=170717_NQ_1K9P&run=2#compress_images) ，其基线质量为85％。 他们测量了用户感知延迟（第一次扫描的时间和总加载时间），发现就总体而言渐进式JPEG可以很好的满足其对低文件大小，可接受的转码/解码时间等要求的方面。
 -  [Facebook为其iOS应用程序提供渐进式JPEG](https://code.facebook.com/posts/857662304298232/faster-photos-in-facebook-for-ios/)。 他们发现数据使用量减少10％，并且显示高质量图像的加载速度提高15％。
 -  [Yelp选择使用渐进式JPEG](https://engineeringblog.yelp.com/2017/06/making-photos-smaller.html) ，发现可以减少约4.5％的图像大小。 他们还使用MozJPEG节省了13.8％的额外费用。

许多图像占比很高的网站，例如 [Pinterest](https://pinterest.com) ，也在产品中使用了渐进式JPEG。

<img
        class="lazyload small"
        data-src="images/book-images/pinterest-loading-large.png"
        alt="Pinterests JPEGs are all progressively encoded. This optimizes the user experience by loading them each scan-by-scan." />

> Pinterest的JPEG都是渐进式编码的。 这通过逐个扫描加载它们来优化用户体验。



### <a id="the-disadvantages-of-progressive-jpegs" href="#the-disadvantages-of-progressive-jpegs">渐进式JPEG的缺点</a>

渐进式JPEG的解码速度可能比基线JPEG慢——甚至长达3倍。 在具有强大CPU的桌面计算机上，这也许不是一个问题，而是在性能资源有限的的移动设备上。 您基本上需要多次解码过程来显示不完整的图层，这些多次传输可能会占用CPU周期。

渐进式JPEG也不总是体积更小。 对于非常小的图像（如缩略图），渐进式JPEG可能比基线JPEG对应的图片体积要大。 然而，对于这样的小缩略图，渐进式渲染方式带来的价值不会很高。

这意味着在决定是否传输渐进式JPEG时，您需要尝试取得文件大小、网络延迟和CPU周期使用之间的正确平衡。



注意：渐进式JPEG（和所有JPEG）有时可以在移动设备上进行硬解码。 它无法改善对内存占用的影响，但它可以消除一些CPU占用问题。但 并非所有Android设备都支持硬件加速，但高端设备以及所有iOS设备是支持的。
一些用户可能认为渐进式的加载方式是一个缺陷，因为很难判断图像是否已经加载完成。 由于每不同用户之间可能会有很大差异，因此需要您对您的用户的体验进行评估。

### <a id="how-to-create-progressive-jpegs" href="#how-to-create-progressive-jpegs">如何创建渐进式JPEG？</a>

例如 [ImageMagick](https://www.imagemagick.org/), [libjpeg](http://libjpeg.sourceforge.net/), [jpegtran](http://jpegclub.org/jpegtran/), [jpeg-recompress](https://github.com/danielgtaylor/jpeg-archive) 和 [imagemin](https://github.com/imagemin/imagemin) 等工具和类库支持导出渐进式JPEG。 如果您有一个现成的图像优化管道，那么可以直接添加渐进式图像加载支持：

```js
const gulp = require('gulp');
const imagemin = require('gulp-imagemin');

gulp.task('images', function () {
    return gulp.src('images/*.jpg')
        .pipe(imagemin({
            progressive: true
        }))
        .pipe(gulp.dest('dist'));       
});
```

大多数图像编辑工具默认将图像保存为基线JPEG文件。

<img
        class="lazyload"
        data-src="images/book-images/photoshop-large.jpg"
        alt="photoshop supports exporting to progressive jpeg from the file export menu" />

> 大多数图像编辑工具默认将图像保存为基线JPEG文件。 您可以通过转到文件 - >导出 - >保存为Web（旧版），然后单击渐进式选项，将您在Photoshop中创建的任何图像保存为渐进式JPEG。 Sketch还支持直接导出渐进式JPEG  - 导出为JPG并在保存图像时选中“渐进式”复选框。</figcaption>



### <a id="chroma-subsampling" href="#chroma-subsampling">色度（或色彩）子采样</a>

（注：Subsampling也可翻译为亚采样，[下采样](<https://www.cnblogs.com/jokerjason/p/9429452.html>)）

我们的眼睛对于图像（色度）中的颜色细节比对亮度（或简称亮度 - 亮度的度量）更容易丢失。 [色度子采样](https://en.wikipedia.org/wiki/Chroma_subsampling) 是一种压缩方式，可降低图像信号中的颜色精度，提高亮度。 这样可以减小文件大小而不会对图像质量产生负面影响，在某些情况下可以降低达 [15-17%](https://calendar.perfplanet.com/2015/why-arent-your-images-using-chroma-subsampling/)，并且是可以用于JPEG图像。 亚采样还可以减少图像内存使用量。



<img
        class="lazyload"
        data-src="images/book-images/luma-signal-large.jpg"
        alt="signal = chroma + luma" />



由于对比度负责形成我们在图像中看到的形状，因此定义它的亮度非常重要。 老式的旧照片或黑白照片不包含颜色，但由于亮度的原因，它们的显示效果可以像它们的颜色一样细腻。 色度（颜色）对视觉感知的影响较小。



<img
        class="lazyload"
        data-src="images/book-images/no-subsampling-large.jpg"
        alt="JPEG includes support for numerous subsampling types: none, horizontal and horizontal and vertical." />

> JPEG支持许多不同的子采样类型：无，水平和水平以及垂直。 该图来自FrédéricKayser的[马蹄蟹的JPEG](http://frdx.free.fr/JPEG_for_the_horseshoe_crabs.pdf) 。



在谈论子采样时，讨论了许多常见的样本。 通常为`4:4:4`，`4:2:2`和`4:2:0`。 但这些代表什么呢？ 假设子样本采用格式A:B:C。 A是一行中的像素数，对于JPEG，这通常是4. B表示第一行中的颜色量，C表示第二行中的颜色。

There are a number of common samples discussed when talking about subsampling. Generally, `4:4:4`, `4:2:2` and `4:2:0`. But what do these represent? Let’s say a subsample takes the format A:B:C. A is the number of pixels in a row and for JPEGs this is usually 4. B represents the amount of color in the first row and C the color in the second.

 -  `4:4:4` 没有压缩，因此颜色和亮度完全被传输。
 -  `4:2:2` 水平半采样，垂直全采样。
 -  `4:2:0` 从第一行像素的一半中采样颜色，忽略第二行。

<aside class="note"><b>Note:</b> jpegtran和cjpeg支持单独的亮度和色度质量配置。 这可以通过添加`-sample`标志来完成（例如`-sample 2x1`）。

一些好的一般规则：子采样（`-sample 2x2`）非常适合照片。 无子采样（`-sample 1x1`）最适用于屏幕截图、横幅和按钮。 最后（`2x1`）在你不确定要使用什么的时候使用。

通过减少色度分量中的像素，可以显着减小颜色分量的大小，最终减小字节大小。

（TODO）



<img
        class="lazyload"
        data-src="images/book-images/subsampling-large.jpg"
        alt="Chrome subsampling configurations for a JPEG at quality 80." />

> 质量为80的JPEG的色度子采样配置。



对于大多数类型的图像，使用色度子采样是值得考虑的。 它也有一些值得注意的例外：由于子采样依赖于我们人眼视觉的限制，因此对于其中颜色细节可能与亮度一样重要（例如医学图像）的压缩图像使用起来并不是很好。

包含字体的图像也会受到影响，因为文本的不良二次取样会降低其易读性。 更锐利的边缘难以使用JPEG压缩，因为它旨在更好地处理具有更柔和过渡的摄影场景。



<img
        class="lazyload small"
        data-src="images/book-images/Screen_Shot_2017-08-25_at_11.06.27_AM-large.jpg"
        alt="Be careful when using heavy subsampling with images containing text" />



> [Understanding JPEG](http://compress-or-die.com/Understanding-JPG/) 一文中建议在处理包含文本的图像时，使用4:4:4(1×1) 的子采样



备注：JPEG规范中未指定色度子采样的确切方法，因此不同的解码器处理它的方式不同。 MozJPEG和libjpeg-turbo使用相同的缩放方法。 较旧版本的libjpeg使用不同的方法来添加颜色中的铃声伪像。（TODO）

Trivia: The exact method of Chroma subsampling wasn’t specified in the JPEG specification, so different decoders handle it differently. MozJPEG and libjpeg-turbo use the same scaling method. Older versions of libjpeg use a different method that adds ringing artifacts in colors.

<aside class="note"><b>Note:</b> 使用“保存为网络图像”的功能时，Photoshop会自动设置色度子采样。 当图像质量设置在51-100之间时，不会使用子采样（`4:4:4`）。 当质量低于此值时，将使用`4:2:0`子采样。 这是当质量从51切换到50时可以显著观察到的文件大小降低的一个原因。</aside>
<aside class="note"><b>Note:</b>在二次抽样讨论中，经常提到术语 [YCbCr](https://en.wikipedia.org/wiki/YCbCr)。 这是一个可以表示伽马校正的 [RGB](https://en.wikipedia.org/wiki/RGB_color_model) 色彩空间的模型。 Y是伽马校正的亮度，Cb是蓝色的色度分量，Cr是红色的色度分量。 当你观察ExifData时，你会看到YCbCr接近采样水平。</aside>
有关色度子采样的进一步阅读，请参考[为什么您的图像不使用色度子采样？](https://calendar.perfplanet.com/2015/why-arent-your-images-using-chroma-subsampling/)

（TODO，该章节涉及了好多图像显示方面的专业数据，翻译粗略的参考谷歌翻译以及部分博客，还需要对术语等进行校对调整）



### <a id="how-far-have-we-come-from-the-jpeg" href="#how-far-have-we-come-from-the-jpeg">我们举例JPEG有多遥远？</a>

**以下是当前网络上图像格式的分布状态：**

*tl;dr – 这非常的分散，我们通常使用不同的现代图像处理技术有选择的对不同的浏览器提供不同的格式支持*



<img
        class="lazyload"
        data-src="images/book-images/format-comparison-large.jpg"
        alt="modern image formats compared based on quality." />

> 不同的现代图像格式（和优化器）用于演示目标文件大小在26KB的可能处理方式。 我们可以使用[SSIM](https://en.wikipedia.org/wiki/Structural_similarity) （结构相似性）或 [Butteraugli](https://github.com/google/butteraugli)来比较图像质量，我们将在后面详细介绍。



（译者注：Butteraugli 是 Google 的一个开源工具，用来评判两个图像之间的相似度。通过识别图像之间一些最受关注的差异点并给出相似度分值。）

*   **[JPEG 2000](https://en.wikipedia.org/wiki/JPEG_2000) (2000)** – 从基于离散余弦的变换到基于小波的方法的JPEG切换的改进。 **浏览器支持：Safari桌面+ iOS**
*   **[JPEG XR](https://en.wikipedia.org/wiki/JPEG_XR) (2009)** – 支持 [HDR](http://wikivisually.com/wiki/High_dynamic_range_imaging) 和[宽色域](http://wikivisually.com/wiki/Gamut) 空间的JPEG和JPEG 2000的替代品。 以稍慢的编码/解码速度生成比JPEG更小的文件。 **浏览器支持：Edge，IE。**
*   **[WebP](https://en.wikipedia.org/wiki/WebP) (2010)** – 谷歌研发的基于块预测的格式，支持有损和无损压缩。 提供类似JPEG的字节存储和类似PNG的透明度支持。由于缺乏色度子采样配置和渐进加载， 解码时间也比JPEG解码慢。**浏览器支持：Chrome，Opera。 通过Safari和Firefox进行实验。**
*   **[FLIF](https://en.wikipedia.org/wiki/Free_Lossless_Image_Format) (2015)** – 声称优于PNG的无损图像格式、无损WebP、无损BPG和基于压缩比的无损JPEG 2000。 **浏览器支持：无。 请注意，有一个[JS浏览器内解码器](https://github.com/UprootLabs/poly-flif)。**
*   **HEIF and BPG.** 从压缩的角度来看，它们是相同的，但具有不同的包装器。
*   **[BPG](https://en.wikipedia.org/wiki/Better_Portable_Graphics) (2015)** – 基于HEVC（[高效视频编码](http://wikivisually.com/wiki/High_Efficiency_Video_Coding)），旨在提高JPEG的压缩效率。 与MozJPEG和WebP相比，似乎可以提供更好的文件大小。 由于许可证问题而不太可能获得广泛的应用。 **浏览器支持：无。 请注意，有一个[JS浏览器内解码器](https://bellard.org/bpg/)。**
*   **[HEIF](https://en.wikipedia.org/wiki/High_Efficiency_Image_File_Format) (2015)** – 用于存储具有约束的帧间可预测的HEVC编码图像的图像和图像序列的格式。 Apple在 [WWDC](https://www.cnet.com/news/apple-ios-boosts-heif-photos-over-jpeg-wwdc/) 上宣布他们将探索在iOS上切换从JPEG到HEIF的使用，理由是文件大小可节省2倍。 **浏览器支持：在撰写本文时没有。 最终，Safari桌面和iOS 11**（TODO）

如果您为了可视化，可能会欣赏上述提供的视觉比较工具。（ [tools one](https://people.xiph.org/~xiphmont/demo/daala/update1-tool2b.shtml) 、 [tools these](http://xooyoozoo.github.io/yolo-octo-bugfixes/#cologne-cathedral&jpg=s&webp=s)）

因此，**浏览器的支持是分散的**，如果您希望利用上述任何一项，您可能需要有条件地为每个目标浏览器提供支持。 在Google，我们已经看到了WebP的一些实现，所以我们很快就会深入探讨它。

您还可以根据浏览器可支持渲染的图像来决定使用.jpg扩展名（或任何其他）提供图像格式（例如WebP，JPEG 2000）的媒体类型支持。 这允许服务器端[内容类型协商](https://www.igvita.com/2012/12/18/deploying-new-image-formats-on-the-web/)决定发送哪个图像而不需要更改HTML。 Instart Logic等服务在向客户提供图像时使用此方法。

接下来，让我们谈谈当您无法有条件地提供不同图像格式时的可选方案：**优化JPEG编码器**。



### <a id="optimizing-jpeg-encoders" href="#optimizing-jpeg-encoders">优化JPEG编码器s</a>

现代JPEG编码器尝试生成更小，更高保真度的JPEG文件，同时保持与现有浏览器和图像处理应用程序的兼容性。 它们避免了在生态系统中引入新的图像格式或更改，来实现压缩增益。。 两个这样的编码器分别是是MozJPEG和Guetzli。

***tl;dr 您应该使用哪个jpeg优化编码器？***

* 一般的Web资产：MozJPEG
* 更关心质量甚于编码时长：使用Guetzli
* 如果您需要可配置性：
 * [JPEGRecompress](https://github.com/danielgtaylor/jpeg-archive) (在MozJPEG的技术长构建) 注：该项目编译时需要引入MozJPEG依赖。
 * [JPEGMini](http://www.jpegmini.com/)。它类似于Guetzli——自动选择最佳质量。虽然技术上不如Guetzli复杂，但速度更快，而且目标是质量范围更适合网络使用。
 * [ImageOptim API](https://imageoptim.com/api) (带有免费的在线界面](https://imageoptim.com/online)) 。它在颜色处理方面是独一无二的。您可以单独选择颜色质量和整体质量。它自动选择色度次采样级别，以保持屏幕截图中的高分辨率颜色，但与此同时还能避免在自然照片中的平滑颜色上浪费字节。




### <a id="what-is-mozjpeg" href="#what-is-mozjpeg">什么是MozJPEG？</a>

Mozilla以 [MozJPEG](https://github.com/mozilla/mozjpeg)的形式提供现代化的JPEG编码器。 它宣城可以减少高达10％的JPEG文件体积。 使用MozJPEG压缩的文件支持跨浏览器工作，它包括逐行扫描优化，网格量化（丢弃最少压缩的细节）和一些优化的量化表预设等常用功能，有助于创建更平滑的高DPI图像（尽管ImageMagick可以实现这一点如果你愿意研究XML配置）。

[ImageOptim](https://github.com/ImageOptim/ImageOptim/issues/45) 支持MozJPEG，并且有一个相对可靠的可配置 [imagemin插件](https://github.com/imagemin/imagemin-mozjpeg) 。 以下是Gulp的示例实现：

```js
const gulp = require('gulp');
const imagemin = require('gulp-imagemin');
const imageminMozjpeg = require('imagemin-mozjpeg');

gulp.task('mozjpeg', () =>
    gulp.src('src/*.jpg')
    .pipe(imagemin([imageminMozjpeg({
        quality: 85
    })]))
    .pipe(gulp.dest('dist'))
);
```



<img
        class="lazyload"
        data-src="images/book-images/Modern-Image10-large.jpg"
        alt="mozjpeg being run from the command-line" />



<img
        class="lazyload"
        data-src="images/book-images/Modern-Image11-large.jpg"
        alt="mozjpeg compression at different qualities. At q=90, 841KB. At q=85, 562KB. At q=75, 324KB. Similarly, Butteraugli and SSIM scores get slightly worse as we lower quality." />



> MozJPEG：不同质量的文件大小和视觉相似度得分的比较。



我使用 [jpeg-archive](https://github.com/danielgtaylor/jpeg-archive) 项目中的 [jpeg-compress](https://github.com/imagemin/imagemin-jpeg-recompress) 来计算源图像的SSIM（结构相似度）分数。 SSIM是一种用于测量两个图像之间的相似性的方法，其中SSIM得分是相对于另一个给定的被认为是“完美的”图像的质量度量。


根据我的经验，MozJPEG是一个很好的选择，可以在高视觉效果的情况下压缩网络图像来减少文件大小。 对于中小尺寸的图像，我发现MozJPEG（质量= 80-85）可以节省30-40％的文件大小，同时保持可接受的SSIM，在jpeg-turbo上提供5-6％的提升。 它确实具有比基线JPEG具有[更高的编码成本](http://www.libjpeg-turbo.org/About/Mozjpeg)，但你可能不会发现显式的阻塞。（注：show stopper可理解为严重程度极高的硬件或[软件错误](https://en.wikipedia.org/wiki/Software_bug)，需要立即修复）

**Note：**如果您需要一个支持MozJPEG的工具以及一些额外的配置支持和一些免费的图像比较工具，请查看 [jpeg-recompress](https://github.com/danielgtaylor/jpeg-archive)。 Web Performance in Action的作者Jeremy Wagner在使用[此配置](https://twitter.com/malchata/status/884836650563579904) 时取得了一些成功。



### <a id="what-is-guetzli" href="#what-is-guetzli">什么是Guetzli？</a>

[Guetzli](https://github.com/google/guetzli) 是一款很有发展情景的慢速可感知的JPEG编码器，谷歌尝试找到最小的JPEG，使得它在感知上与人眼无法区分。 它执行一系列测试，产生最终JPEG的方案，解释每个方案中心理视觉偏差。最终它选择得分最高的方案作为最终输出。

为了测量图像之间的差异，Guetzli使用[Butteraugli](https://github.com/google/butteraugli)，一种基于人类感知测量图像差异的模型（下面讨论）。 Guetzli可以考虑其他JPEG编码器没有的一些视觉属性。 例如，在所看到的绿光量和对蓝色的敏感度之间存在关系，因此可以稍微不那么精确地编码绿色附近的蓝色变化。

<aside class="note"><b>Note:</b>图像文件大小**更多地取决于质量**的选择而不是**编解码器**的选择。与通过切换编解码器实现的文件大小节省相比，最低和最高质量JPEG之间的文件大小差异要大得多。 使用最低的可接受质量非常重要。 如果非必要，请避免将质量设置得过高。 </aside>
与其他压缩方式相比，Guetzli[宣称](https://research.googleblog.com/2017/03/announcing-guetzli-new-open-source-jpeg.html ) 对于给定的Butteraugli分数，图像的数据大小减少了20-30％。 对于使用Guetzli的一个严重的警告是它非常非常慢，目前仅适用于静态内容。 从README，我们可以注意到Guetzli需要大量内存 - 每百万像素可能需要1分钟+ 200MB RAM。 在这个GitHub的一个[issue](https://github.com/google/guetzli/issues/50)中，有一个关于Guetzli实际体验的准确描述。 当您在静态站点的构建过程中对图像进行优化时，它是理想的选择，但在按需执行时则不太理想。

<aside class="note"><b>Note:</b>作为静态站点的构建过程中优化图像时，Guetzli可能更适合。或者不用按需执行图像优化的情况。 </aside>
像ImageOptim这样的工具支持Guetzli优化（在[最新版本](https://imageoptim.com/)中）。

```js
const gulp = require('gulp');
const imagemin = require('gulp-imagemin');
const imageminGuetzli = require('imagemin-guetzli');

gulp.task('guetzli', () =>
    gulp.src('src/*.jpg')
    .pipe(imagemin([
        imageminGuetzli({
            quality: 85
        })
    ]))
    .pipe(gulp.dest('dist'))

);
```



<img
        class="small lazyload"
        data-src="images/book-images/Modern-Image12-large.jpg"
        alt="guetzli being run from gulp for optimisation" />



使用Guetzli编码3 x 3MP图像来节省存储空间需要差不多7分钟（以及高CPU使用率）。 为了存档更高分辨率的照片，我能看出它有很重要的价值。



<img
        class="lazyload"
        data-src="images/book-images/Modern-Image13-large.jpg"
        alt="comparison of guetzli at different qualities. q=100, 945KB. q=90, 687KB. q=85, 542KB." />

> Guetzli：不同质量的文件大小和视觉相似度得分的比较。



**Note：**建议在高质量图像上运行Guetzli（例如，未压缩的输入图像，PNG源或100％质量或接近的JPEG）。虽然它也可以用于其他图像（例如质量为84或更低的JPEG），但结果可能更差。

虽然使用Guetzli压缩图像非常（非常）耗时并且会使您的风扇疯狂旋转，但对于较大的图像，这是值得的。 我已经看到了许多例子，在保持视觉逼真度的同时，它可以在任何地方保存高达40%的文件大小。 这使其非常适合存档照片。 在中小尺寸的图像上，我仍然看到了一些节省（在10-15KB范围内），但它们并没有那么明显。 Guetzli可以在压缩时在较小的图像上引入更多的液化扭曲。

您可能还对Eric Portis研究感兴趣，将Guetzli与Cloudinary的自动压缩进行[比较](https://cloudinary.com/blog/a_closer_look_at_guetzli) ，以获得有效的不同数据点。



### <a id="mozjpeg-vs-guetzli" href="#mozjpeg-vs-guetzli">MozJPEG与Guetzli相比如何？</a>

比较不同的JPEG编码器很复杂程度，需要比较压缩图像的质量和保真度以及最终尺寸。 正如图像压缩专家KornelLesiński指出的那样，对这些方面中的一个而非两个方面进行基准测试可能会导致[无效](https://kornel.ski/faircomparison)的结论。

Guetzli与MozJPEG相比如何？  -  Kornel的观点：

 -  Guetzli更关心获得更高质量的图像（据说`q=90+` 最适合，但是MozJPEG的最佳点是`q=75`左右）
 -  Guetzli的压缩速度要慢得多（两者都产生标准的JPEG，所以解码像往常一样快）
 -  MozJPEG不会自动选择质量设置，但您可以使用外部工具找到最佳质量，例如： [jpeg-archive](https://github.com/danielgtaylor/jpeg-archive)

存在许多方法用于确定压缩图像在视觉上是否与其图像源相似或可视效果相似。 图像质量研究通常使用 [SSIM](https://en.wikipedia.org/wiki/Structural_similarity)（结构相似性）等方法。 然而，Guetzli对Butteraugli进行了优化。



### <a id="butteraugli" href="#butteraugli">Butteraugli</a>

[Butteraugli](https://github.com/google/butteraugli) 是Google的一个项目，它估计一个人可能注意到两个图像的视觉图像退化（心理视觉相似性）的时间点。它为那些在几乎看不到差异的领域中可靠的图像打分。Butteraugli不仅给出了一个标量分数，而且还计算了一个不同级别的空间地图。当ssim查看来自图像的错误集合时，Butteraugli查看的是最糟糕的部分。



<img
        class="lazyload small"
        data-src="images/book-images/Modern-Image14-medium.jpg"
        alt="butteraugli validating an image of a parrot" />



> 上图是一个使用Butteraugli找到JPEG最小质量阈值的示例，处理之后视觉降低非常严重，用户甚至可以注意到某些区域并不太清楚。但是它使得文件总大小减少了65％。</figcaption>



在实践中，您需要为视觉质量定义一个目标，然后运行一系列不同的图像优化策略，查看您的Butteraugli分数，然后选择最适合文件大小和质量级别之间的平衡。

<img
        class="lazyload small"
        data-src="images/book-images/Modern-Image15-large.jpg"
        alt="butteraugli being run from the command line" />

> 总而言之，安装Bazel后，大约花费了我30分钟，在本地安装了ButoGLI，并获得了一个C++源用于构建，以便在我的Mac上能正确编译。使用它是非常直接的：指定要比较的两个图像（源和压缩版本），它会给你一个分数。



**Butteraugli与其他比较视觉相似性的方法有何不同？**

Guetzli项目成员的这一[评论](https://github.com/google/guetzli/issues/10#issuecomment-276295265)表明，Guetzli在Butteraugli上的得分最高，在ssim和mozjpeg上的得分最低，两者都差不多。这与我在自己的图像优化策略中所做的研究是一致的。我在图像上运行Butteraugli和一个节点模块，比如[img-ssim](https://www.npmjs.com/package/img-ssim) ，将源代码与guetzli和mozjpeg前后的ssim得分进行比较。

**组合编码器？**

对于更大的图像，我发现在mozjpeg（jpegtran，而不是cjpeg）中将guetzli与**无损压缩**结合在一起可以进一步减少10-15%的文件大小（综合来说是55%），而ssim的减少非常小。这是我需要注意的，需要实验和分析，但也已经被其他领域的人，如 [Ariya Hidayat](https://ariya.io/2017/03/squeezing-jpeg-images-with-guetzli) 进行了尝试并得到了确认的的结果。

Mozjpeg是一个初学者友好的Web资产编码器，速度相对较快，生成的图像质量较好。由于Guetzli是资源密集型的，在更大、更高质量的图像上效果最好，因此我将为中级到高级用户保留此选项。



## <a id="what-is-webp" href="#what-is-webp">什么是WebP?</a>

[WebP](https://developers.google.com/speed/webp/)是Google最近推出的一种图像格式，旨在以可接受的视觉质量提供较低的文件大小，用于无损和有损压缩。它包括对alpha通道透明度和动画的支持。

在过去的一年中，webp在有损和无损模式下压缩比提高了几个百分点，在速度方面，算法的速度提高了两倍，在解压方面提高了10%。WebP并不是一个全能的工具，但它在图像压缩社区中有一定的地位和不断增长的用户基础。我们来研究一下原因。



<img
        class="lazyload"
        data-src="images/book-images/Modern-Image16-large.jpg"
        alt="comparison of webp at different quality settings. q=90, 646KB. q=80= 290KB. q=75, 219KB. q=70, 199KB" />
> WebP：不同质量的文件大小和视觉相似性得分的比较。</figcaption>



### <a id="how-does-webp-perform" href="#how-does-webp-perform">WebP的表现如何？</a>

**有损压缩**

WebP团队提到使用VP8或VP9视频关键帧编码变体的WebP有损文件大小比JEPG文件降低约 [25-34%](https://developers.google.com/speed/webp/docs/webp_study) 。

在低质量范围（0-50）中，WebP具有超过JPEG的巨大优势，因为它可以模糊出丑陋的块状伪影。 中等质量设置（-m 4 -q 75）是默认的平衡速度/文件大小。 在较高范围（80-99），WebP的优势缩小。如果速度比质量更重要，则建议使用 WebP。

// todo 这个ugly block 该怎么翻译合适一些

**无损压缩**

[WebP无损文件比PNG文件小26%](https://developers.google.com/speed/webp/docs/webp_lossless_alpha_study)。与 PNG 相比，无损加载时间减少 3%。也就是说，您通常不希望在 Web 上为用户提供无损的图像。无损和锐化边缘（例如非 JPEG）之间存在差异。无损 WebP 可能更适合存档内容。

**透明度**

WebP 具有无损的 8 位透明度通道，仅比 PNG 多 22% 的字节。它还支持有损的 RGB 透明度，这是 WebP 独有的功能。

**元数据**

WebP 文件格式支持EXIF 照片元数据和XMP 数字文档元数据。它还包含ICC颜色配置文件。

WebP 以占用更多 CPU 的成本提供更好的压缩。早在 2013 年，WebP 的压缩速度比 JPEG 慢约10倍，但现在可以忽略不计（某些图像可能慢 2倍）。对于作为生成一部分处理的静态图像，这应该不是大问题。动态生成的图像可能会出现可感知的 CPU 开销，并且需要评估。

**备注：**WebP有损质量设置与JPEG无法直接比较。 “70％质量”的JPEG与“70％质量”的WebP图像完全不同，因为WebP通过丢弃更多数据来实现更小的文件大小。



### <a id="whos-using-webp-in-production" href="#whos-using-webp-in-production">谁在生产中使用WebP？</a>

许多大公司在生产中使用WebP来降低成本并减少网页加载时间。

谷歌报告称，使用WebP比其他有损压缩方案节省30-35％，每天提供430亿个图像请求，其中26％是无损压缩。 这是大量的请求以及效果显著的节省。 如果[浏览器支持](http://caniuse.com/#search=webp)更好，更广泛，节省的开销无疑会更大。 Google还在Google Play和YouTube等制作网站中使用它。

Netflix、Amazon、Quora、Yahoo、Walmart、eBay、Guardian、Fortune和USA Today，都通过WebP为支持它的浏览器压缩和服务图像。VoxMedia通过为他们的Chrome用户切换到WebP，将[1-3秒的加载时间](https://product.voxmedia.com/2015/8/13/9143805/performance-update-2-electric-boogaloo)降到了极限。当切换到为Chrome用户提供服务时，[500px](https://iso.500px.com/500px-color-profiles-file-formats-and-you/) 的图像文件大小平均减少了25%，图像质量类似或更好。

除了这个样本列表中指出的还有不少公司。



<img
        class="small lazyload"
        data-src="images/book-images/webp-conversion-large.jpg"
        alt="WebP stats at Google: over 43B image requests a day" />



> Google的WebP使用：每天在YouTube，Google Play，Chrome数据保护程序和G +上提供每天430亿次WebP图像请求。



### <a id="how-does-webp-encoding-work" href="#how-does-webp-encoding-work">WebP编码如何工作？</a>

WebP的有损编码旨在与JPEG静止图像竞争。 WebP的有损编码有三个关键阶段：

**Macro-blocking** - 将图像拆分为 16×16（宏）的亮度（luma）像素块和两个 8×8 块色度（chroma）像素。这在 JPEG 执行色彩空间转换、色度通道向下采样和图像细分的思路中可能听起来很熟悉。

//todo  注：这里的Macro-blocking暂时没想好一个比较专业的术语，就先按原样写下来，



<img
        class="small lazyload"
        data-src="images/book-images/Modern-Image18-medium.png"
        alt="Macro-blocking example of a Google Doodle where we break a range of pixels down into luma and chroma blocks."/>



**预测** - 宏块（Macro-blocking）的每个4×4子块都应用了一个有效进行筛选的预测模型。 这定义了一个块周围的两组像素 -  A（它正上方的行）和L（它左边的列）。 使用这两个编码器填充具有4×4像素的测试块并确定哪个创建最接近原始块的值。 Colt McAnlis在[WebP有损模式的工作原理](https://medium.com/@duhroach/how-webp-works-lossly-mode-33bd2b1d0670)中更深入地讨论了这一点



<img
        class="lazyload very-small"
        data-src="images/book-images/Modern-Image19-small.png"
        alt="Google Doodle example of a segment displaying the row, target block and column L when considering a prediction model."/>



应用离散余弦变换（DCT），其步骤类似于JPEG编码。 关键的区别在于使用[算术压缩器](https://www.youtube.com/watch?v=FdMoL3PzmSA&index=7&list=PLOU2XLYxmsIJGErt5rrCqaSGTMyyqNt2H) 和 JPEG的Huffman编码。

如果您想深入了解，Google Developer的文章 [WebP 压缩技术](https://developers.google.com/speed/webp/docs/compression)将深入探讨这一主题。



### <a id="webp-browser-support" href="#webp-browser-support">WebP 浏览器支持</a>

但并非所有浏览器都支持WebP根据[canius.com](https://caniuse.com/#search=webp)，全球用户支持率约为74%。Chrome和Opera在本地支持它。Safari、Edge和Firefox已经对此进行了试验，但还没有正式发布。这通常会将获取WebP图像的任务留给Web开发人员。以后再谈。

以下是每个主要的浏览器和支持信息：

* Chrome：Chrome在23版本开始全力支持。
* Chrome for Android：从Chrome 50开始
* Android：从Android 4.2开始
* Opera：从12.1开始
* Opera Mini：所有版本
* Firefox：一些测试版支持
* Edge：一些测试版支持
* Internet Explorer：不支持
* Safari：一些测试版支持

WebP并非没有缺点。 它缺乏全分辨率色彩空间选项，不支持渐进式解码。 也就是说，WebP是不错的浏览器支持工具，但在撰写本文时仅限于Chrome和Opera，可能会覆盖足够多的用户，因此它值得考虑作为备选方案。



### <a id="how-do-i-convert-to-webp" href="#how-do-i-convert-to-webp">我如何将图像转为WebP?</a>

一些商业和开源图像编辑处理包支持WebP。 其中一个特别有用的应用是XnConvert：一个免费的，跨平台的批量图像处理转换器。

**备注：**避免将低质量或普通质量的JPEG转换为WebP这一点非常重要。使用 JPEG压缩工具生成WebP图像是一个常见的错误。 这可能导致WebP效率降低，因为它必须保存图像和JPEG添加的失真，从而导致两次损失质量。适合 Feed转换应用程序的是高质量源文件，最好是原始文件。

**[XnConvert](http://www.xnview.com/en/xnconvert/)**

XnConvert支持批量图像处理，兼容500多种图像格式。 您可以组合80多个单独的操作，以多种方式转换或编辑图像。



<img
        class="small lazyload"
        data-src="images/book-images/Modern-Image20-large.png"
        alt="XNConvert app on Mac where a number of images have been converted to WebP"
         />



> XnConvert支持批量图像优化，支持从源文件到WebP和其他格式的直接转换。 除压缩外，XnConvert还可以帮助进行元数据分离，裁剪，颜色深度定制和其他变换。</figcaption>



xnview网站上列出的一些选项包括：

*   元数据：编辑
*   变换：旋转，裁剪，调整大小
*   调整：亮度，对比度，饱和度
*   过滤器：模糊，浮雕，锐化
*   效果：遮蔽，水印，晕影（暗角）

（注：关于Vignetting的意思，推荐下[What is Vignetting?](https://photographylife.com/what-is-vignetting)）

您的操作结果可以导出为大约70种不同的文件格式，包括WebP。 XnConvert适用于Linux，Mac和Windows。 强烈建议使用XnConvert，特别是对于小型企业。

**Node modules**

[Imagemin](https://github.com/imagemin/imagemin) 是一种流行的图像缩小模块，它还具有用于将图像转换为WebP的附加组件 ([imagemin-webp](https://github.com/imagemin/imagemin-webp))。支持无损和有损两种模式。

要安装imagemin和imagemin-webp，请执行以下命令：

```
> npm install --save imagemin imagemin-webp
```

然后我们可以使用require()导入两个模块，并对项目目录中的任何图像（例如JPEG）使用。 下面我们使用有损编码，WebP编码器质量为60：


```js
const imagemin = require('imagemin');
const imageminWebp = require('imagemin-webp');

imagemin(['images/*.{jpg}'], 'images', {
    use: [
        imageminWebp({quality: 60})
    ]
}).then(() => {
    console.log(‘Images optimized’);
});
```

与 JPEG 类似，在我们的输出中可以注意到压缩伪影。评估设置为什么质量对您自己的图像有价值。Imagemin-webp 还可用于通过将`lossless: true`参数传递给选项来编码无损质量 WebP 图像（支持 24 位颜色和完全透明度）：


```js
const imagemin = require('imagemin');
const imageminWebp = require('imagemin-webp');

imagemin(['images/*.{jpg,png}'], 'build/images', {
    use: [
        imageminWebp({lossless: true})
    ]
}).then(() => {
    console.log(‘Images optimized’);
});
```


Sindre Sorhus 构建在imagemin-webp和 [WebPack的WebP加载器](https://www.npmjs.com/package/webp-loader) 的[Gulp WebP插件](https://github.com/sindresorhus/gulp-webp) 也是可用的available。Gulp插件接受imagemin插件所做的任何选项：

```js
const gulp = require('gulp');
const webp = require('gulp-webp');

gulp.task('webp', () =>
    gulp.src('src/*.jpg')
    .pipe(webp({
        quality: 80,
        preset: 'photo',
        method: 6
    }))
    .pipe(gulp.dest('dist'))
);
```

或无损：

```js
const gulp = require('gulp');
const webp = require('gulp-webp');

gulp.task('webp-lossless', () =>
    gulp.src('src/*.jpg')
    .pipe(webp({
        lossless: true
    }))
    .pipe(gulp.dest('dist'))
);
```

**使用Bash进行批量图像优化**

XNConvert支持批量图像压缩，但如果您希望避免使用应用程序或构建系统，则bash和图像优化工具可以使事情变得非常简单。

您可以使用[cwebp](https://developers.google.com/speed/webp/docs/cwebp) 将图片批量转换为WebP：

```
find ./ -type f -name '*.jpg' -exec cwebp -q 70 {} -o {}.webp \;
```

或通过使用 [jpeg-recompress](https://github.com/danielgtaylor/jpeg-archive) 批量优化你的源图像：

```
find ./ -type f -name '*.jpg' -exec jpeg-recompress {} {} \;
```

并使用[svgo ](https://github.com/svg/svgo)修改这些SVG（稍后我们会介绍）：

```
find ./ -type f -name '*.svg' -exec svgo {} \;
```

Jeremy Wagner在[使用Bash进行图像优化](https://jeremywagner.me/blog/bulk-image-optimization-in-bash) 方面有更全面的内容，另外还有一篇关于[并行工作](https://jeremywagner.me/blog/faster-bulk-image-optimization-in-bash) 的文章也值得阅读。

**其他WebP图像处理和编辑应用程序包括：**

   * Leptonica  - 一个完整的开源图像处理和分析网站

* Sketch支持直接输出到WebP
* GIMP  - 免费的，开源的Photoshop替代品。 图像编辑器。
* ImageMagick  - 创建，编写，转换或编辑位图图像。 免费使用的命令行应用程序。
* Pixelmator  - 适用于Mac的商业图像编辑器。
* Photoshop的WebP插件 - 来自谷歌的可免费使用图像导入和导出插件。

**Android**：您可以使用Android Studio将现有的BMP，JPG，PNG或静态GIF图像转换为WebP格式。 有关更多信息，请参阅[使用Android Studio创建WebP图像](https://developer.android.com/studio/write/convert-webp.html)。

### <a id="how-do-i-view-webp-on-my-os" href="#how-do-i-view-webp-on-my-os">如何在我的操作系统上查看WebP图像？</a>

虽然您可以将WebP图像拖放到基于Blink内核的浏览器（Chrome，Opera，Brave）进行预览，但您也可以使用Mac或Windows的附加组件直接从操作系统中预览它们。

几年前[Facebook尝试使用WebP](https://www.cnet.com/news/facebook-tries-googles-webp-image-format-users-squawk/)，并发现尝试右键单击照片并将其保存到磁盘的用户不会在浏览浏览器之外查看，由于是使用了WebP。 这里有三个关键问题：
 - “另存为”，但无法在本地查看WebP文件。 这是通过Chrome将自己注册为“.webp”处理程序来解决的。
 - “另存为”，然后将图片附加到电子邮件中，并与没有Chrome的人共享。 Facebook通过在用户界面中引入一个醒目的“下载”按钮并在用户请求下载时返回JPEG来解决这个问题。
 - 右键单击 -> 复制URL  - > 在Web上共享URL。 这是通过[内容类型协商](https://www.igvita.com/2012/12/18/deploying-new-image-formats-on-the-web/)解决的

这些问题对您的用户可能不太重要，但顺便说一下，这是一个有关社交可共享性的有趣说明。值得庆幸的是，现在在不同操作系统上存在用于查看和使用WebP的应用程序。

在Mac上，尝试WebP的[快速查看插件](https://github.com/Nyx0uf/qlImageSize) （qlImageSize）。 它工作得很好：



<img
        class="small lazyload"
        data-src="images/book-images/Modern-Image22-large.jpg"
        alt="Desktop on a mac showing a WebP file previewed using the Quick Look plugin for WebP files"
         />



在Windows上，您还可以下载[WebP编解码器软件包](https://storage.googleapis.com/downloads.webmproject.org/releases/webp/WebpCodecSetup.exe) ，以便在文件资源管理器和Windows照片查看器中预览WebP图像。



### <a id="how-do-i-serve-webp" href="#how-do-i-serve-webp">我如何提供WebP服务？</a>

没有WebP支持的浏览器最终可能根本不显示图像，这并不理想。 为了避免这种情况，我们可以使用一些策略来基于浏览器支持有条件地提供WebP服务。



<img
        class="lazyload"
        data-src="images/book-images/play-format-webp-large.jpg"
        alt="The Chrome DevTools Network panel displaying the waterfall for the Play Store in Chrome, where WebP is served."
         />



> Chrome的开发者工具网络面板在“teype”列下突出显示提供给基于Blink的浏览器的WebP文件。



<img
        class="lazyload small"
        data-src="images/book-images/play-format-type-large.jpg"
        alt="While the Play store delivers WebP to Blink, it falls back to JPEGs for browsers like Firefox."
         />



> 虽然Play商店向Blink传送WebP，但对于像Firefox这样的浏览器，它会回归到JPEG。




以下是从服务器向用户提供WebP图像的一些选项：

**Using .htaccess to Serve WebP Copies**

以下是当服务器上存在匹配的.webp版本的JPEG / PNG文件时，如何使用.htaccess文件向支持的浏览器提供WebP文件。

Vincent Orback推荐这种方法：

浏览器可以通过[Accept Header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept)显式地发出[WebP支持信号](http://vincentorback.se/blog/using-webp-images-with-htaccess/)。 如果您控制后端，则可以返回图像的WebP版本（如果图像存在于磁盘上）而不是JPEG或PNG格式。 但这并不总是可行的（例如对于像GitHub Page或S3这样的静态地址），所以在考虑此选项之前一定要检查。

以下是Apache Web服务器的示例.htaccess文件：

```
<IfModule mod_rewrite.c>

  RewriteEngine On

  # Check if browser support WebP images
  RewriteCond %{HTTP_ACCEPT} image/webp

  # Check if WebP replacement image exists
  RewriteCond %{DOCUMENT_ROOT}/$1.webp -f

  # Serve WebP image instead
  RewriteRule (.+)\.(jpe?g|png)$ $1.webp [T=image/webp,E=accept:1]

</IfModule>

<IfModule mod_headers.c>

    Header append Vary Accept env=REDIRECT_accept

</IfModule>

AddType  image/webp .webp
```

如果页面上出现.webp图像存在问题，请确保在服务器上启用了image/webp MIME类型。

Apache：将以下代码添加到.htaccess文件中：

```
AddType image/webp .webp
```

Nginx：将以下代码添加到mime.types文件中：

```
image/webp webp;
```

**注意**：Vincent Orback有一个示例[phtaccess配置](https://github.com/vincentorback/WebP-images-with-htaccess)用于提供WebP以供参考，Ilya Grigorik维护了一组非常有用的用于提供[WebP的配置脚本](https://github.com/igrigorik/webp-detect) 。

**使用 `<picture>` 标签**

浏览器本身能够通过使用`<picture>`标签来选择要显示的图像格式。 `<picture>`标签使用多个`<source>`元素，带有一个`<img>`标签，它是包含图像的实际DOM元素。 浏览器循环浏览源并检索第一个匹配项。 如果用户的浏览器不支持`<picture>`标记，则呈现`<div>`并使用`<img>`标记。

**注意**：注意`<source>`的位置，因为顺序很重要。 不要在传统格式之后放置`image/webp`资源，而是将它们放在之前。 能解释它的浏览器将使用它们，而那些不能解释它的浏览器将转而更广泛支持的框架上。 如果图像的物理尺寸相同（不使用`media`属性），也可以按文件大小的顺序放置图像。 通常，这与将遗留最后的顺序相同。

这是一些示例HTML：

```html
<picture>
  <source srcset="/path/to/image.webp" type="image/webp">
  <img src="/path/to/image.jpg" alt="">
</picture>

<picture>   
    <source srcset='paul_irish.jxr' type='image/vnd.ms-photo'>  
    <source srcset='paul_irish.jp2' type='image/jp2'>
    <source srcset='paul_irish.webp' type='image/webp'>
    <img src='paul_irish.jpg' alt='paul'>
</picture>

<picture>
   <source srcset="photo.jxr" type="image/vnd.ms-photo">
   <source srcset="photo.jp2" type="image/jp2">
   <source srcset="photo.webp" type="image/webp">
   <img src="photo.jpg" alt="My beautiful face">
</picture>
```

**CDN自动转换为WebP**

一些CDN支持自动转换为WebP，并且可以使用[客户端提示](http://cloudinary.com/documentation/responsive_images#automating_responsive_images_with_client_hints)尽可能地提供WebP图像。 请与您的CDN联系，了解他们的服务中是否包含WebP支持。 你可能有一个简单的解决方案等着你。

**WordPress WebP支持**

**Jetpack**  -  Jetpack，一个流行的WordPress插件，包括一个名为[Photon](https://jetpack.com/support/photon/)的CDN图像服务。 使用Photon，您可以获得无缝的WebP图像支持。 Photon CDN包含在Jetpack的免费级别中，因此这是一个很好的方案，并且是一个非入侵式的实现。 缺点是Photon会调整您的图像大小，在您的URL中放置一个查询字符串，并且每个图像都需要额外的DNS查找。

**Cache Enabler and Optimizer** — 如果您使用的是WordPress，则至少有一个半开放选项。 开源插件 [Cache Enabler](https://wordpress.org/plugins/cache-enabler/)有一个菜单复选框选项，用于缓存要提供的WebP图像，如果WebP图像可用并且当前用户的浏览器支持它们。 这使得提供WebP图像变得容易。 有一个缺点：Cache Enabler需要使用名为Optimizer的姐妹程序，该程序需要支付年费。 对于真正的开源解决方案而言，这似乎不合时宜。

**Short Pixel** — 与Cache Enabler一起使用的另一个选项（也需付费）是Short Pixel。Short Pixel功能与上文描述的Optimizer非常类似。您每月可以免费优化多达 100 个图像。



**压缩动画GIF以及为什么`<video>`更好**

动画GIF继续广泛使用，尽管格式非常有限。 虽然从社交网络到流行媒体网站的所有内容都大量嵌入动画GIF，但格式*从未*专为视频存储或动画而设计。 事实上，[GIF89a规范](https://www.w3.org/Graphics/GIF/spec-gif89a.txt) 指出GIF并不打算作为动画的平台。 [颜色数量，帧数和尺寸](http://gifbrewery.tumblr.com/post/39564982268/can-you-recommend-a-good-length-of-clip-to-keep-gifs) 都会影响动画GIF大小。 切换到视频节省了最多。



<img
        class="lazyload"
        data-src="images/book-images/animated-gif-large.jpg"
        alt="Animated GIF vs. Video: a comparison of file sizes at ~equivalent quality for different formats."
         />

> 动画GIF vs. Video: 不同格式的等效质量的文件大小比较。



**提供与MP4视频相同的文件通常可以减少文件大小80％及以上**。 GIF不仅经常浪费大量带宽，而且加载时间更长，包含更少的颜色，并且通常提供不完整的用户体验。 您可能已经注意到上传到Twitter的动画GIF在Twitter上比在其他网站上表现更好。 [Twitter上的动画GIF实际上不是GIF](http://mashable.com/2014/06/20/twitter-gifs-mp4/#fiiFE85eQZqW)。为了改善用户体验并减少带宽消耗，上传到Twitter的动画GIF实际上转换为视频。 同样，[Imgur会在上传时将GIF转换为视频](https://thenextweb.com/insider/2014/10/09/imgur-begins-converting-gif-uploads-mp4-videos-new-gifv-format/)，然后将其静默转换为MP4。

（注：原文说的是 sub-part user experiences，想说的说就是部分用户体验，故翻译为不完整的用户体验。）

为什么GIF要大很多倍？ 动画GIF将每个帧存储为无损GIF图像 - 是的，竟然是无损的。 我们经常遇到的低质量是由于GIF仅限于256色调色板。 格式通常很大，因为与H.264等视频编解码器不同，它不考虑用于压缩的间隔帧。 MP4视频将每个关键帧存储为有损JPEG，丢弃一些原始数据以实现更好的压缩。

//译者注：H.264会将连续视频分为关键帧和前后可预测的非关键帧来减少文件大小。

//todo 添加一些说明

**如果您可以切换到视频**

*   使用 [ffmpeg](https://www.ffmpeg.org/) 将动画GIF（或源）转换为H.264 MP4。我是用来自[Rigor](http://rigor.com/blog/2015/12/optimizing-animated-gifs-with-html5-video) 的命令：
`ffmpeg -i animated.gif -movflags faststart -pix_fmt yuv420p -vf "scale=trunc(iw/2)*2:trunc(ih/2)*2" video.mp4`
*   ImageOptim API also supports [converting animated gifs to WebM/H.264 video](https://imageoptim.com/api/ungif), [removing dithering from GIFs](https://github.com/kornelski/undither#examples) which can help video codecs compress even more.
*   ImageOptim API还支持将[GIF动画转换为WebM / H.264视频](https://imageoptim.com/api/ungif)、[从GIF中删除抖动](https://github.com/kornelski/undither#examples)，这可以帮助视频编解码器压缩更多。

**如果您必须使用动画GIF**

*   像Gifsicle这样的工具可以剥离元数据、未使用的调色板条目，并最小化帧之间的变化
*   Consider a lossy GIF encoder. The [Giflossy](https://github.com/kornelski/giflossy) fork of Gifsicle supports this with the `—lossy` flag and can shave ~60-65% off size. There’s also a nice tool based on it called [Gifify](https://github.com/vvo/gifify). For non-animated GIFs, convert them to PNG or WebP.
*   考虑有损GIF编码器。 fork于Gifsicle 的 [Giflossy](https://github.com/kornelski/giflossy)支持用`-lossy`标志，可以减少约60-65％的尺寸。 还有一个很好的基于它的工具，叫做 [Gifify](https://github.com/vvo/gifify)。 对于非动画GIF，请将它们转换为PNG或WebP。

有关更多信息，请查看Rigor的[Book of GIF](https://rigor.com/wp-content/uploads/2017/03/TheBookofGIFPDF.pdf)。



## <a id="svg-optimization" href="#svg-optimization">SVG优化</a>

保持SVG精简意味着剥离任何不必要的东西。 使用编辑器创建的SVG文件通常包含大量冗余信息（元数据，注释，隐藏层等）。 通常可以安全地删除此内容或将其转换为更小的形式，而不会影响正在呈现的最终SVG。




<img
        class="lazyload small"
        data-src="images/book-images/Modern-Image26-large.jpg"
        alt="svgo"
         />



> [SVGOMG](https://jakearchibald.github.io/svgomg/), by Jake Archibald, 是一个GUI界面，通过选择优化，并输出实时预览状态，您可以根据自己的喜好优化SVG 。



**SVG优化的一些一般规则（SVGO）：**

*   缩小并压缩您的SVG文件。 SVG实际上只是用XML表示的文本内容，如CSS，HTML和JavaScript，应该缩小和压缩以提高性能。
* 使用预定义的SVG形状而不是路径，如`<rect>`，`<circle>`，`<ellipse>`，`<line>`和`<polygon>`。 优先选择预定义的形状会减少生成最终图像所需的标记量，这意味着浏览器解析和栅格化的代码更少。 降低SVG复杂性意味着浏览器可以更快地显示它。（译者注：栅格化也存在于PS之中，大致是说将一个向量类型的形状、路径转换为一个点阵类型的图像。形状、路径是可以自由缩放不收影响）
*   如果必须使用路径，请尝试减少曲线和路径。 尽可能简化并组合它们。 Illustrator的[简化工具](http://jlwagner.net/talks/these-images/#/2/10)擅长删除复杂艺术作品中的多余点，同时消除不规则性。（//todo irregularities）
*   避免使用群组。 如果做不到，试着简化它们。
*   删除不可见的图层。
*   避免任何Photoshop或Illustrator效果。 它们可以转换为大型光栅图像。
*   仔细检查任何不支持SVG的嵌入式光栅图像
* 使用工具优化SVG。  [SVGOMG](https://jakearchibald.github.io/svgomg/)是Jake Archibald为[SVGO](https://github.com/svg/svgo)提供的一个非常方便的基于Web的GUI，我发现它非常好用。 如果使用Sketch，则可以在导出时使用[SVGO的Sketch插件](https://www.sketchapp.com/extensions/plugins/svgo-compressor/)来缩小文件大小。



<img
        class="lazyload small"
        data-src="images/book-images/svgo-precision-large.jpg"
        alt="svgo precision reduction can sometimes have a positive impact on size"
         />



> 通过SVGO以高质量模式运行SVG源（尺寸减少29％）与低质量模式（尺寸减少38％）的示例。



[SVGO](https://github.com/svg/svgo) 是一种基于节点的SVG优化工具。 SVGO可以通过降低`<path>`定义中数字的精度来减小文件大小。 小数点后的每一个数字都会增加一个字节，这就是为什么*更改精度*（位数）会严重影响文件大小的原因。 要非常小心地改变精度，因为它可以很直观的看到你的形状受到影响。




<img
        class="lazyload"
        data-src="images/book-images/Modern-Image28-large.jpg"
        alt="where svgo can go wrong, oversimplifying paths and artwork"
         />



> 重要的是要注意虽然SVGO在前面的示例中表现良好而没有过度简化路径和形状，但是在很多情况下可能并非如此。 观察上述火箭上的灯条，在较低的精度下出现了扭曲。



**在命令行使用SVGO：**

SVGO可以作为[全局npm CLI](https://www.npmjs.com/package/svgo)安装，如果您更喜欢GUI：

```
npm i -g svgo
```

然后可以对本地SVG文件运行，如下所示：

```
svgo input.svg -o output.svg
```

它支持您可能期望的所有选项，包括调整浮点精度：

```
svgo input.svg --precision=1 -o output.svg
```

有关支持的选项的完整列表，请参阅SVGO的 [readme](https://github.com/svg/svgo)。

**别忘了压缩SVG！**



<img
        class="lazyload"
        data-src="images/book-images/before-after-svgo-large.jpg"
        alt="before and after running an image through svgo"
         />



另外，不要忘记[使用Gzip压缩](https://calendar.perfplanet.com/2014/tips-for-optimising-svg-delivery-for-the-web/)您的SVG资源或使用Brotli提供服务。 由于它们是基于文本的，因此它们的压缩效果非常好（约占原始来源的50％）。

当Google发布新logo时，我们宣布它的[最小版本](https://twitter.com/addyosmani/status/638753485555671040)只有305字节。

（译者注：Brotli也是Google提供的一种新的压缩工具，比Gzip压缩效率更高，可以在Nginx代理的时候替换gzip模块使用，具体如何配置可以看我的另一篇文章）



<img
        class="lazyload very-small"
        data-src="images/book-images/Modern-Image30-large.jpg"
        alt="the smallest version of the new google logo was only 305 bytes in size"
         />



有许多[高级SVG技巧](https://www.clicktorelease.com/blog/svg-google-logo-in-305-bytes/)可以用来进一步减少（一直到146字节）！ 可以说，无论是通过工具还是通过手动，你可以多花一些东西来精简你的SVG。

**SVG Sprites技术**

SVG 对于图标来说可能[功能强大](https://css-tricks.com/icon-fonts-vs-svg/) ，它提供了一种将可视化表示为子画面的方法，而无需图标字体这种[古怪解决方法](https://www.filamentgroup.com/lab/bulletproof_icon_fonts.html)。它有比图标字体（SVG笔画属性），更好的定位控制（无需破解伪元素和CSS`display`）和SVG[更容易访问](http://www.sitepoint.com/tips-accessible-svg/)更精细的CSS样式控制。

//todo 下一段先不删了，得先搞明白svg sprite的的原理。mark：[SVG Sprites技术介绍](https://www.zhangxinxu.com/wordpress/2014/07/introduce-svg-sprite-technology/)

像svg-sprite和IcoMoon这样的工具，可以通过使用CSS Sprite，Symbol Sprite或Stacked Sprite自动将SVG组合成Sprite。 Una Kravets对如何使用gulp-svg-sprite进行SVG Sprite工作流程进行了实用的描述，值得一试。 Sara Soueidan还介绍了如何在她的博客上从图标字体过渡到SVG。

Tools like [svg-sprite](https://github.com/jkphl/svg-sprite) and [IcoMoon](https://icomoon.io/) can automate combining SVGs into sprites which can be used via a [CSS Sprite](https://css-tricks.com/css-sprites/), [Symbol Sprite](https://css-tricks.com/svg-use-with-external-reference-take-2) or [Stacked Sprite](http://simurai.com/blog/2012/04/02/svg-stacks). Una Kravets has a practical [write-up](https://una.im/svg-icons/#💁) on how to use gulp-svg-sprite for an SVG sprite workflow worth checking out. Sara Soueidan also covers [making the transition from icon fonts to SVG](https://www.sarasoueidan.com/blog/icon-fonts-to-svg/) on her blog.

**更多阅读**

Sara Soueidan的[优化网络SVG交付技巧](https://calendar.perfplanet.com/2014/tips-for-optimising-svg-delivery-for-the-web/)和Chris Coyier的[实用SVG书](https://abookapart.com/products/practical-svg)非常好。 我还发现Andreas Larsen的优化SVG帖子很有启发性[第1部分](https://medium.com/larsenwork-andreas-larsen/optimising-svgs-for-web-use-part-1-67e8f2d4035)，[部分 2](https://medium.com/larsenwork-andreas-larsen/optimising-svgs-for-web-use-part-2-6711cc15df46)。[在Sketch中准备和导出SVG图标](https：// medium .com / sketch-app-sources / preparation-and-exports-svg-icons-in-sketch-1a3d65b239bb)也是一本很棒的读物。



## <a id="avoid-recompressing-images-lossy-codecs" href="#avoid-recompressing-images-lossy-codecs">避免使用有损编解码器重新压缩图像</a>

建议始终从原始图像压缩。 重新压缩图像会产生影响。 假设您拍摄的质量为60的JPEG已被压缩。如果使用有损编码重新压缩此图像，则会显得更糟。 每一轮额外的压缩都将引入代际损失 - 信息将丢失，压缩冗余将开始积累。 即使您在高质量设置下重新压缩。

（译者注：类似JPEG这种有损的格式，经过DCT变换和量化过程就会损失精度，所以即使设置高质量保存，也需要引入额外的冗余信息保持精度，所以对同一个JPEG图片重复压缩不但对图像有影响，还会显著增加文件体积。）

为了避免这种陷阱，**首先要设置您愿意接受的最低质量**，并且从一开始就可以节省最多的文件大小。 然后，您可以避免此陷阱，因为仅从质量降低来减少任何文件大小都会看起来很糟糕。

重新编码有损文件几乎总会给你一个较小的文件，但这并不意味着你可以从中获得尽可能多的质量。



<img
        class="lazyload"
        data-src="images/book-images/generational-loss-large.jpg"
        alt="generational loss when re-encoding an image multiple times"
         />



> 上面，根据Jon Sneyers的精彩[视频](https://www.youtube.com/watch?v=w7vXJbLhTyI) 和后续[文章](http://cloudinary.com/blog/why_jpeg_is_like_a_photocopier) ，我们可以看到使用多种格式重新压缩的代际损失影响。 如果从社交网络保存（已压缩）图像并重新上传它们（导致重新压缩），则可能遇到此问题。 质量损失将会增加。



由于网格量化，MozJPEG（可能是偶然的）对再压缩降级具有更好的抵抗力。 它不是精确地压缩所有DCT值，而是检查+ 1 / -1范围内的接近值，以查看相似的值是否压缩到更少的位。有损flif有一个类似于有损png的黑客程序，在（重新）压缩之前，它可以查看数据并决定丢弃什么。 重新压缩的PNG具有可以检测到的“漏洞”，以避免进一步改变数据。

**编辑源文件时，请以PNG或TIFF等无损格式存储它们**，这样您就可以保留尽可能多的质量。 您的构建工具或图像压缩服务处理输出您向用户提供的压缩版本，而质量损失最小。



## <a id="reduce-unnecessary-image-decode-costs" href="#reduce-unnecessary-image-decode-costs">减少不必要的图像解码并裁剪消耗</a>

我们之前已经为我们的用户提供了比我们用户所需的大、更高分辨率的图像。 这需要付出代价。 对于平均水平的移动硬件上的浏览器来说，解码和调整图像大小是高消耗的操作。 如果使用CSS或width/height属性发送大型图像并重新缩放，您可能会看到这种情况发生，并且可能会非常影响性能。



<img
        class="lazyload"
        data-src="images/book-images/image-pipeline-large.jpg"
        alt="There are many steps involved in a browser grabbing an image specified in a tag and displaying it on a screen. These include request, decode, resize, copy to GPU and display."
         />



> 当浏览器加载图像时，它必须将图像从原始源格式（例如JPEG）解码为存储器中的BitMap（位图）。 通常需要调整图像的大小（例如，宽度已设置为其容器的百分比）。 解码和调整图像大小非常消耗资源，并且可能会延迟显示图像所需的时间。
>
> （译者注：BitMap在安卓中也是一种数据类型）



发送浏览器可以渲染的图像，而无需调整大小是理想的选择。因此，利用 [`srcset`和 `sizes`](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images) ，为目标屏幕尺寸和分辨率提供最小的图像 - 我们将很快介绍`srcset`。

忽略图像上的`width`或`height`属性也会对性能产生负面影响。 如果没有它们，浏览器会为图像指定一个较小的占位符区域，直到有足够的字节到达它以便知道正确的尺寸。 此时，文档布局必须在回调中进行更新。



<img
        class="lazyload small"
        data-src="images/book-images/devtools-decode-large.jpg"
        alt="image decode costs shown in the chrome devtools"
         />



> 浏览器必须经过许多步骤才能在屏幕上绘制图像。 除了获取它们之外，还需要解码图像并经常调整大小。 这些事件可以在Chrome DevTools的中 [Timeline](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/performance-reference)的进行审计。



较大的图像也会增加内存大小的成本。 解码图像是每像素约4个字节。 稍不注意就会使得浏览器崩溃; 在低端设备上，开始内存交换并不需要那么多。 因此，请密切关注图像解码，调整大小和内存成本。



<img
        class="lazyload small"
        data-src="images/book-images/image-decoding-mobile-large.jpg"
        alt="Decoding images can be incredibly costly on average and lower-end mobile hardware"
         />



> 在普通以及低端手机上解码图像的成本非常高。 在某些情况下，解码速度可能会慢5倍（如果不会更长的话）。



在构建新的[移动网络体验](https://medium.com/@paularmstrong/twitter-lite-and-high-performance-react-progressive-web-apps-at-scale-d28a00e780a3)时，Twitter通过确保为用户提供适当大小的图像来提高图像解码性能。 Twitter的timeline上显示大量图像的解码时间从大约400毫秒降低到约19毫秒！




<img
        class="lazyload"
        data-src="images/book-images/image-decoding-large.jpg"
        alt="Chrome DevTools Timeline/Performance panel highlighting image decode times before and after Twitter Lite optimized their image pipeline. Before was higher."
         />



> Chrome DevTools Timeline/Performance 面板使用绿色突出显示Twitter Lite优化其图像通道之前和之后的图像解码时间对比。



### <a id="delivering-hidpi-with-srcset" href="#delivering-hidpi-with-srcset">使用`srcset`提供HiDPI图像</a>

用户可以通过一系列具有高分辨率屏幕的移动或桌面设备访问您的网站。 [设备像素比率](https://stackoverflow.com/a/21413366) （DPR）（也称为“CSS像素比率”）决定了CSS如何解释设备的屏幕分辨率。 DPR由手机制造商创建，旨在提高移动屏幕的分辨率和清晰度，而不会使元素显得太小。

为了匹配用户可能期望的图像质量，请向其设备提供最合适的分辨率图像。 可以将锐化的高DPR图像（例如2×，3×）提供给支持它们的设备。 低级或标准DPR图像应该在没有高分辨率屏幕的情况下提供给用户，因为这样的2× 图像通常会产生更多的字节。

<img
        class="lazyload"
        data-src="images/book-images/device-pixel-ratio-large.jpg"
        alt="A diagram of the device pixel ratio at 1×, 2× and 3×. Image quality appears to sharpen
        as DPR increases and a visual is shown comparing device pixels to CSS pixels."
         />



> 设备像素比率：许多网站都会跟踪常用设备的DPR，包括[material.io](https://material.io/devices/)和[mydevice.io](https://mydevice.io/devices/).



[srcset](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images) 允许浏览器为每个设备选择最佳可用图像，例如 为2×移动显示器选择2×图像。 没有`srcset`支持的浏览器可以降级到`<img>`标签中指定的默认`src`。

```
<img srcset="paul-irish-320w.jpg,
             paul-irish-640w.jpg 2x,
             paul-irish-960w.jpg 3x"
     src="paul-irish-960w.jpg" alt="Paul Irish cameo">
```

Image CDNs like [Cloudinary](http://cloudinary.com/blog/how_to_automatically_adapt_website_images_to_retina_and_hidpi_devices) and [Imgix](https://docs.imgix.com/apis/url/dpr) both support controlling image density to serve the best
density to users from a single canonical source.

像[Cloudinary](http://cloudinary.com/blog/how_to_automatically_adapt_website_images_to_retina_and_hidpi_devices)和[Imgix](https://docs.imgix.com/apis/url/dpr)这样的图像CDN都支持控制图像密度，以提供最佳密度给来源符合规范的用户。

**Note**：您可以在此免费的 [Udacity](https://www.udacity.com/course/responsive-images--ud882) 课程和Web基础知识的[图像指南](https://developers.google.com/web/fundamentals/design-and-ui/responsive/images)中了解有关设备像素比率和响应式图像的更多信息。

一个友好的提示， [Client Hints](https://www.smashingmagazine.com/2016/01/leaner-responsive-images-client-hints/) 还可以提供一种替代方法，在响应式图像标记中指定每个可能的像素密度和格式。 相反他们会将此信息附加到HTTP请求，以便Web服务器可以选择最适合当前设备的屏幕密度。



### <a id="art-direction" href="#art-direction">艺术方向</a>

虽然向用户提供正确的解决方案很重要，但有些网站还需要从**[艺术方向](http://usecases.responsiveimages.org/#art-direction)**考虑这一点。 如果用户位于较小的屏幕上，您可能需要裁剪或放大并显示主题以充分利用可用空间。 尽管艺术方向超出了本文的范围，但像[Cloudinary](http://cloudinary.com/blog/automatically_art_directed_responsive_images%20) 这样的服务提供的API可以尽可能地尝试自动化。



<img
        class="lazyload"
        data-src="images/book-images/responsive-art-direction-large.jpg"
        alt="responsive art direction in action, adapting to show more or less of an image in a cropped manner depending on device"
         />



> 艺术指导：埃里克·波蒂斯（Eric Portis）整理了一幅优秀的[样本](https://ericportis.com/etc/cloudinary/)，展示了如何将响应式图像用于艺术指导。 此示例调整主要根据图像在不同断点处的视觉特征，以充分利用可用空间。



## <a id="color-management" href="#color-management">色彩管理</a>

至少有三种不同的颜色视角：生物学，物理学和印刷学。 在生物学中，色彩是一种[感性现象](http://hubel.med.harvard.edu/book/ch8.pdf)。 物体反射不同波长组合的光。 我们眼中的光感受器将这些波长转化为我们称之为颜色的感觉。 在物理学中，重要的是光 - 频率和亮度。 打印更多的是关于色轮，墨水和艺术模型。（//todo 专业数据）

理想情况下，世界上的每个屏幕和Web浏览器都会显示完全相同的颜色。 不幸的是，由于一些固有的不一致性，他们没有。 色彩管理使我们能够通过颜色模型，空间和轮廓来显示颜色。

（译者注：不一致性受硬件制造水印限制，色域、位深都能造成视觉上的偏差）

#### 色彩模型

[色彩模型](https://en.wikipedia.org/wiki/Gamma_correction)是用于从较小的一组原色生成完整颜色范围的系统。 有不同类型的色彩空间使用不同的参数来控制色彩。 一些色彩空间的控制参数比其他色彩空间少 - 例如 灰度仅具有用于控制黑色和白色之间的亮度的单个参数。

两种常见的颜色模型是加法模型和减法模型。 加色模型（如RGB，用于数字显示）使用光来显示颜色，而减色模型（如CMYK，用于打印）通过消除光线来工作。

（译者注：CMYK常用于喷绘打印，其中包括黑色或金色等由于制备工艺原因等无法由其他颜色很好的配比而成，因而需要单独配置，也被称为专色。）



<img
        class="lazyload small"
        data-src="images/book-images/colors_ept6f2-large.jpg"
        alt="sRGB, Adobe RGB and ProPhoto RGB" />

> 在RGB红色，绿色和蓝色光以不同的组合添加，以产生广泛的颜色。 CYMK（青色，品红色，黄色和黑色）通过不同颜色的墨水从白纸中减去亮度。



[了解颜色模型和专色系统](https://www.designersinsights.com/designer-resources/understanding-color-models/)可以很好地描述其他颜色模型和模式，例如HSL，HSV和LAB。



#### 色彩空间

[色彩空间](http://www.dpbestflow.org/color/color-space-and-color-profiles#space)是可以为给定图像表示的特定颜色范围。 例如，如果图像包含多达1670万种颜色，则不同的颜色空间允许使用这些颜色的更窄或更宽的色彩范围。 一些开发人员将颜色模型和颜色空间称为相同的东西。

[sRGB](https://en.wikipedia.org/wiki/SRGB) 旨在成为Web的[标准色彩空间](https://www.w3.org/Graphics/Color/sRGB.html)，基于RGB。 这是一个小的色彩空间，通常被认为是最低的共同点的集合，是跨浏览器色彩管理最安全的选择。 其他色彩空间（如 [Adobe RGB](https://en.wikipedia.org/wiki/Adobe_RGB_color_space)或 [ProPhoto RGB](https://en.wikipedia.org/wiki/ProPhoto_RGB_color_space)  - 用于Photoshop和Lightroom）可以代表比sRGB更鲜艳的色彩，但后者在大多数网络浏览器，游戏和显示器中更为普遍，它通常是需要被关注的。



<img
        class="lazyload small"
        data-src="images/book-images/color-wheel_hazsbk-large.jpg"
        alt="sRGB, Adobe RGB and ProPhoto RGB" />





> 在上面我们可以看到色域的可视化 - 色彩空间可以定义的颜色范围。



色彩空间有三个通道（红色，绿色和蓝色）。 在8位模式下，每个通道可以有255种颜色，使我们总共有1670万种颜色。 16位图像可以显示数万亿种颜色。



<img
        class="lazyload small"
        data-src="images/book-images/srgb-rgb_ntuhi4-large.jpg"
        alt="sRGB, Adobe RGB and ProPhoto RGB" />





> 使用 [Yardstick](https://yardstick.pictures/tags/img%3Adci-p3)中的图像比较sRGB，Adobe RGB和ProPhoto RGB。 当您无法看到不能被显示的颜色时，在sRGB中显示此概念非常困难。 除了大多数饱和的丰富颜色外，sRGB与宽色域的常规照片应该具有相同的一切。（译者注：原文的juicy color，直白的翻译是多汁的颜色，应该是用于说明颜色种类非常丰富。todo）




色彩空间之间的差异（如sRGB，Adobe RGB和ProPhoto RGB）是它们的色域（它们可以用阴影再现的颜色范围），光源和[伽玛曲线](http://blog.johnnovak.net/2016/09/21/what-every-coder-should-know-about-gamma/)。 sRGB比Adobe RGB小约20％，ProPhoto RGB比Adobe RGB大[约50％](http://www.petrvodnakphotography.com/Articles/ColorSpace.htm) 。 上面的图像源来自 [Clipping Path](http://clippingpathzone.com/blog/essential-photoshop-color-settings-for-photographers)

[宽色域](http://www.astramael.com/) 是描述色域大于sRGB的色域的术语。 这些类型的显示器正变得越来越普遍。 也就是说，许多数字显示器仍然无法显示明显优于sRGB的颜色配置文件。 在Photoshop中保存网页图像时，请考虑使用“转换为sRGB”选项，除非定位具有高端宽色域屏幕的用户。

**Note：**使用原始照片时，请避免使用sRGB作为主要色彩空间。 它比大多数相机支持的色彩空间小，并且可能导致剪裁。 相反，在导出Web时，可以在更大的色彩空间（如ProPhoto RGB）进行处理并输出到sRGB。



**是否存在宽色域对Web上的内容有意义的情况？**

是的。 如果图像包含非常饱和/丰富/鲜艳的颜色，并且您关心它在支持它的屏幕上同样的效果。 但是，在很少发生的真实照片中。 通常很容易调整颜色以使其看起来充满活力，而实际上并不超过sRGB色域。

那是因为人类的颜色感知并不是绝对的，而是相对于我们周围的环境而且容易被欺骗。 如果您的图像包含高亮的荧光色，那么使用宽色域查看时会更容易。



#### 伽马校正和压缩

[Gamma校正](https://en.wikipedia.org/wiki/Gamma_correction) （或仅Gamma）控制图像的整体亮度。 更改伽玛还可以改变红色与绿色和蓝色的比例。 没有伽玛校正的图像可能看起来像是漂白或太暗。

在视频和计算机图形中，伽玛被用于压缩，类似于数据压缩。 这允许您以较少的位（8位而不是12位或16位）压缩有用的亮度级别。 人类对亮度的感知与物理光量不成线性比例。 在为人眼编码图像时，以真实物理形式表示颜色将是浪费的。 伽玛压缩用于在更接近人类感知的尺度上编码亮度。

对于伽玛压缩，有用的亮度范围符合8位精度（大多数RGB颜色使用0-255）。 所有这一切都来自如下事实：如果颜色使用与物理学具有1：1关系的一些单位，则RGB值将从1到百万，其中值0-1000看起来不同，但是999000-1000000之间的值看起来相同。 想象一下，在一个只有一支蜡烛的黑暗房间里。 点亮第二根蜡烛，你会发现室内光线亮度明显增加。 添加第三根蜡烛，它看起来更加明亮。 现在想象一下，在一个有100根蜡烛的房间里。 点燃第101个蜡烛，第102个蜡烛。 你不会注意到亮度的变化。

即使在两种情况下，物理上都添加了完全相同的光量。 因此，当光线明亮时，眼睛不那么敏感，伽马压缩会“压缩”明亮的值，因此在物理方面，亮度水平不太精确，但是人体调整了比例，因此从人的角度来看，所有值都同样精确。

**Note：**此处的伽马压缩/校正与您在Photoshop中可能配置的图像伽玛曲线不同。当伽玛压缩按预期工作时，将看不错任何区别。



#### 颜色配置文件

颜色配置文件是描述设备色彩空间的信息。 它用于在不同的色彩空间之间进行转换。 配置文件尝试确保图像在这些不同类型的屏幕和介质上看起来尽可能相似。

图像具有可嵌入的颜色配置文件，如[国际色彩联盟](http://www.color.org/icc_specs2.xalter)（ICC）所描述的，以精确地表示颜色应该如何显示。 这是由不同的格式支持，包括JPEG，PNG，SVG和WebP](https://developers.google.com/speed/webp/docs/riff_container)，大多数主流浏览器支持嵌入式ICC配置文件。 当图像在应用程序中显示并且它知道显示器的功能时，可以根据颜色配置文件调整这些颜色。

（译者注：JPEG在使用CMYK色彩模式作为喷绘打印的图像时，ICC配置文件就非常重要，尤其是使用代码生成时，否则会导致目标文件色彩显示错误（就是能显示，但是颜色偏差非常大）导致无法打印。以JAVA为例：需要先解析MetaTree，将ICC配置文件嵌入进去，再进行合并。）

**Note：**某些显示器具有与sRGB类似的颜色配置文件，并且无法显示更好的配置文件，因此根据您目标用户的显示设备，嵌入它们的价值可能有限。 确定目标用户是哪些。

嵌入的颜色配置文件也会大大增加图像的大小（偶尔会增加100KB），因此请小心嵌入。 像ImageOptim这样的工具实际上会在找到它们时[自动](https://imageoptim.com/color-profiles.html)删除颜色配置文件。 相反，在缩小尺寸名称中删除ICC配置文件时，浏览器将被迫在显示器的色彩空间中显示图像，这可能导致预期饱和度和对比度的差异。 评估这之间的权衡对您的用例很有意义。

如果您有兴趣了解有关配置文件的更多信息，那么[Nine Degrees Below](https://ninedegreesbelow.com/photography/articles.html) 拥有一套优秀的ICC配置文件颜色管理资源。



#### 颜色配置文件和Web浏览器

早期版本的Chrome对色彩管理没有很好的支持，但2017年使用 [Color Correct Rendering](https://groups.google.com/a/chromium.org/forum/#!topic/blink-dev/ptuKdRQwPAo)进行了改进。 不是sRGB（较新的MacBook Pro）的显示器会将sRGB中的颜色按照显示配置文件进行转换。 这意味着不同系统和浏览器的颜色看起来应该更相似。 Safari，Edge和Firefox现在也可以考虑ICC配置文件，因此具有不同颜色配置文件（例如ICC）的图像现在可以正确显示它们，无论您的屏幕是否具有宽色域。

**Note：**有关色彩如何在网络上工作这方面的更广泛的意见，请参阅Sarah Drasner撰写的[网站色彩指南](https://css-tricks.com/nerds-guide-color-web/)。



## <a id="image-sprites" href="#image-sprites">Image spriting</a>

[Image sprites](https://developers.google.com/web/fundamentals/design-and-ui/responsive/images#use_image_sprites) (或CSS sprites)在 Web 上具有悠久的历史，所有浏览器都支持这些图片，并且通过将图像组合到切片的单个较大图像中来减少页面加载的图像数量，这已成为一种常用方法。

（译者注：Image spriting最好不需要翻译，直译就是图像精灵，反而不好懂，是一种图像切片技术。可参考[知乎回答](https://www.zhihu.com/question/51438507)）



<img
        class="lazyload small"
        data-src="images/book-images/i2_2ec824b0_1-large.jpg"
        alt="Image sprites are still widely used in large, production sites, including the Google homepage."
         />



> mage sprites技术仍然广泛用于大型生产网站，包括谷歌主页。



在HTTP/1.x下，一些开发人员使用spriting来减少HTTP请求。 这带来了许多好处，但是当你突然遇到缓存失效的异常时需要小心 - 对Image sprites的任何一小部分的更改都会使用户缓存中的整个图像无效。

然而，Spriting现在可以是 [HTTP/2](https://hpbn.co/http2/) 下的一个反模式。 使用HTTP/2，最好[加载单个图像](https://deliciousbrains.com/performance-best-practices-http2/)，因为现在可以在单个连接中进行多个请求。 衡量这样做是否适用于您自己的网络设置。



## <a id="lazy-load-non-critical-images" href="#lazy-load-non-critical-images">延迟加载非关键图像</a>

延迟加载是一种Web性能模式，它会延迟浏览器中图像的加载，直到用户需要查看它为止。 例如，滚动时，图像会根据需要异步加载。 这可以进一步提高您从“图像压缩策略”看到的字节节省的数量。

（译者注：避免了非必要的图像加载以节省带宽，尤其是在大量图像场景下性能提升明显，例如图片直播等。）



<img
        class="lazyload"
        data-src="images/book-images/scrolling-viewport-large.jpg"
        alt="lazy-loading images"
         />



必须出现在“首屏”上方或首次出现网页的图像需要立即加载。 然而，屏幕下方的图像对于用户来说是不可见的。 它们不必立即加载到浏览器中。 它们可以在以后加载 - 或者延迟加载 - 只有当用户向下滚动并且有必要显示它们时才可以加载它们。

浏览器本身尚未支持延迟加载（尽管过去曾有过[讨论](https://discourse.wicg.io/t/a-standard-way-to-lazy-load-images/1153/10)）。 相反，我们可以使用JavaScript来添加此功能。



**为什么延迟加载如此有效？**

这种只在必要时加载图像的“延迟”方式有很多好处：

* **减少数据消耗：**由于您没有假设用户需要提前获取所有图像，因此您只需加载最少量的资源。 这总是一件好事，特别是在具获取数据又限制的移动设备上。
* **减少电池消耗**：减少用户浏览器的工作量，从而节省电池寿命。
* **提高下载速度**：将图像繁重的网站上的整个页面加载时间从几秒减少到几乎没有，这对用户体验起到了巨大的推动作用。 事实上，这可能是用户留下来享受您的网站或离开您的网站之间的统计数据区别。

**但是像所有工具一样，强大的功能带来了巨大的责任。**

**避免在首屏上方加载延迟图像。**将其用于长图像列表（例如产品列表）或用户头像列表。 不要将它用作主页横幅。 从技术上和对人的感知上来说，延迟加载fold上方的图像会使加载明显变慢。 它会杀死浏览器的预加载器，渐进式加载和JavaScript会为浏览器带来额外的开销。

（译者注： hero image意为主页横幅）

**滚动时要非常小心延迟加载图像。**如果你等到用户滚动可能会先看到占位符，并且最终会获得图像，如果它们还没有滚过它们的话。 一个建议是在fold上方图像加载后就开始延迟加载，这样加载所有图像而不依赖于用户交互。

**谁在使用延迟加载？**

有关延迟加载的示例，请查看承载大量图像的大多数主要站点。 一些值得注意的网站是[Medium](https://medium.com/)和[Pinterest](https://www.pinterest.com/)。



<img
        class="lazyload"
        data-src="images/book-images/Modern-Image35-large.jpg"
        alt="inline previews for images on medium.com"
         />

> Medium.com上图像的高斯模糊内联预览的示例



许多站点（例如Medium）显示一个小的高斯模糊的内联预览（几个100字节），一旦获取后它就会转换（延迟加载）到一个完整质量的图像。

JoséM.Pérez撰写了关于如何使用[CSS过滤器](https://jmperezperez.com/medium-image-progressive-loading-placeholder/) 实现Medium的这种效果并尝试使用[不同图像格式](https://jmperezperez.com/webp-placeholder-images/)来支持此类占位符的文章。 Facebook还对其著名的200字节方法进行了记录，为这些占位符提供了值得阅读的[封面照片](https://code.facebook.com/posts/991252547593574/the-technology-behind-preview-photos/)。 如果您是Webpack用户，[LQIP加载器](https://lqip-loader.firebaseapp.com/)可以帮助自动完成部分工作。

实际上，您可以搜索自己喜欢的高分辨率照片源站，然后向下滚动页面。 几乎在所有情况下，您都会体验到网站一次只加载几张全分辨率图像，其余图像是占位符颜色或图像。 在继续滚动时，占位符图像将替换为全分辨率图像。 这就是延迟加载的作用。

最近一直在进行的一项技术是*基于矢量*而非基于像素的低质量图像预览，由Tobias Baldauf在他的工具 [SQIP](https://github.com/technopagan/sqip)中进行试验。 这种方法利用实用程序[Primitive](https://github.com/fogleman/primitive)生成SVG预览，该预览由几个简单的形状组成，这些形状近似于目标图像中可见的主要特征，使用 [SVGO](https://github.com/svg/svgo)优化SVG，最后为其添加高斯模糊滤波器; 生成SVG占位符，大小仅为800-1000字节，在所有屏幕上看起来都很清晰，并提供了图像内容的视觉提示。延迟加载和低质量图像预览都可以明显地[结合](https://calendar.perfplanet.com/2017/progressive-image-loading-using-intersection-observer-and-sqip/)起来。

（译者注：crisp原意为脆，结合文章表示图像看起来清晰锐利。）



**如何将延迟加载应用到我的页面？**

有许多技术和插件可用于延迟加载。 我推荐Alexander Farkas的 [lazysizes](https://github.com/aFarkas/lazysizes) ，因为它具有良好的性能、功能、与 [Intersection Observer](https://developers.google.com/web/updates/2016/04/intersectionobserver)的可选集成、以及对插件的支持。

**我该如何使用Lazysizes?**

Lazysizes是一个JavaScript库。 它不需要配置。 下载压缩的js文件并将其包含在您的网页中。

以下是从README文件中获取的一些示例代码：

将class:“lazyload”与data-src或data-srcset属性一起添加到images/iframe中。

您也可以选择使用低质量图像添加src属性：

```html
<!-- non-responsive: -->
<img data-src="image.jpg" class="lazyload" />

<!-- responsive example with automatic sizes calculation: -->
<img
    data-sizes="auto"
    data-src="image2.jpg"
    data-srcset="image1.jpg 300w,
    image2.jpg 600w,
    image3.jpg 900w" class="lazyload" />

<!-- iframe example -->

<iframe frameborder="0"
    class="lazyload"
    allowfullscreen=""
    data-src="//www.youtube.com/embed/ZfV-aYdU4uE">
</iframe>
```

对于本书的Web版本，我将Lazysizes（尽管您可以使用任何替代方案）与Cloudinary配对，以实现按需响应式图像。 这使我可以自由地尝试不同的尺度，质量，格式值以及是否以最小的努力逐步加载：

<img
        class="lazyload"
        data-src="images/book-images/cloudinary-responsive-images-large.jpg"
        alt="Cloudinary supports on-demand control of image quality, format and several other features."
         />

**Lazysizes功能包括：**

* 自动检测当前和后续lazyload元素的可见性更改
 - 包括标准的响应式图像支持（图片和srcset）
 - 为媒体查询功能添加自动大小计算和别名
 - 可以在CSS和JS重页面或Web应用程序上使用数百个images/ iframe
 - 可扩展：支持插件
 - 轻量级但成熟的解决方案
 - 改进了SEO：不会隐藏爬虫的图像/资产

**更多延迟加载的选项**

Lazysizes 不是你唯一的选项，这里还有更多延迟加载的类库：

*   [Lazy Load XT](http://ressio.github.io/lazy-load-xt/)
*   [BLazy.js](https://github.com/dinbror/blazy) (or [Be]Lazy)
*   [Unveil](http://luis-almeida.github.io/unveil/)
*   [yall.js (Yet Another Lazy Loader)](https://github.com/malchata/yall.js) which is ~1KB and uses Intersection Observer where supported.

**延迟加载的问题是什么？**

*   屏幕阅读器，一些搜索机器人和禁用JavaScript的任何用户将无法查看延迟加载JavaScript的图像。 然而，这是我们可以使用`<noscript>`备用方案来解决的问题。
*   滚动侦听器（例如用于确定何时加载延迟加载的图像）可能会对浏览器滚动性能产生负面影响。 它们可能会导致浏览器重绘多次，从而减慢页面进程的滚动速度 - 但是，智能延迟加载库将通过限制来缓解此问题。 一种可能的解决方案是Intersection Observer,，它由lazysizes支持。

延迟加载图像是一种用于减少带宽，降低成本和改善用户体验的普适模式。 评估您的体验是否合理。 有关进一步阅读，请参阅[延迟加载图像](https://jmperezperez.com/lazy-loading-images/)和[实现Medium的渐进式加载](https://jmperezperez.com/medium-image-progressive-loading-placeholder/)。




## <a id="display-none-trap" href="#display-none-trap">避免display:none陷阱</a>

较旧的响应式图像解决方案误解了浏览器在设置CSS的`display`属性时如何处理图像请求。 这可能导致请求的图像明显多于您可能期望的图像，而且另一个原因是`<picture>`和`<img srcset>`才是加载响应图像的首选。

你有没有写过一个媒体类型的请求，并在某些断点处将图像设置为`display：none`？

```html
<img src="img.jpg">
<style>
@media (max-width: 640px) {
    img {
        display: none;
    }
}
</style>
```

或者使用`display：none`类来切换隐藏的图像？

```html
<style>
.hidden {
  display: none;
}
</style>
<img src="img.jpg">
<img src=“img-hidden.jpg" class="hidden">
```

A quick check against the Chrome DevTools network panel will verify that images hidden using these approaches still get fetched, even when we expect them not to be. This behavior is actually correct per the embedded resources spec.、

对Chrome DevTools的network面板进行快速检查，将验证使用这些方法隐藏的图像是否仍会被取出？即使我们预计不会这样。 根据嵌入式资源规范，此行为实际上是正确的。



<img
        class="lazyload"
        data-src="images/book-images/display-none-images-large.jpg"
        alt="Images hidden with display:none still get fetched"
         />



**`display：none`是否会触发对图像`src`的请求？**

```html
<div style="display:none"><img src="img.jpg"></div>
```

指定的图像仍会被请求。 类库不能依赖display：none，因为在JavaScript改变src之前就会请求图像。

**`display：none`是否会触发对`background：url()`的请求？**

```html
<div style="display:none">
  <div style="background: url(img.jpg)"></div>
</div>
```

 只要解析元素，就不会获取CSS背景。 使用`display：none`计算元素子元素的CSS样式不太有用，因为它们不会影响文档的呈现。 子元素的背景图像不会计算或下载。

Jake Archibald的[Request Quest](https://jakearchibald.github.io/request-quest/) 对于使用`display：none`进行响应式图像加载的缺陷进行了很好的测验。 如果对特定浏览器如何处理图像请求加载有疑问，请打开他们的DevTools并自行验证。

同样，尽可能使用`<picture>`和`<img srcset>`而不是依赖于`display：none`。



## <a id="image-processing-cdns" href="#image-processing-cdns">图像处理CDN对您有意义吗？</a>

*您花在阅读博客文章以设置自己的图像处理管道和调整配置所花费的时间通常是远大于服务费。随着[Cloudinary](http://cloudinary.com/)提供免费服务，[Imgix](https://www.imgix.com/)免费试用和[Thumbor](https://github.com/thumbor/thumbor)作为OSS的替代品，有很多选项供您自动使用。*

要实现最佳页面加载时间，您需要优化图像加载。 此优化需要响应式图像策略，并且可以受益于服务器上的图像压缩，以自动选择最佳格式和响应式调整大小。 重要的是，您要尽可能快地以正确的分辨率将正确大小的图像传送到适当的设备。 这样做并不像人们想象的那么容易。

**使用自己的服务器 vs CDN**

由于图像处理的复杂性和不断变化的性质，我们将提供具有该领域经验的人的报价，然后继续提出建议。

“如果你的产品不涉及图像处理，那就不要自己动手了。像Cloudinary [或imgix、Ed]这样的服务比你更有效率且能提供更好地服务，所以请使用它们。如果你担心成本的话，请务必考虑在开发和维护中的开销，以及托管、存储和交付的成本。“   — [Chris Gmyr](https://medium.com/@cmgmyr/moving-from-self-hosted-image-service-to-cloudinary-bd7370317a0d)

目前，我们将同意并建议您考虑使用CDN来满足您的图像处理需求。 下面将根据我们之前提出的任务列表，来对两个CDN进行比较。

**Cloudinary 和 imgix**

Cloudinary和[imgix](https://www.imgix.com/)是两个已创建的图像处理CDN公司。 它们是全球数十万开发商和公司的选择，包括Netflix和Red Bull。 让我们更详细地看一下它们。

**基础是什么?**

除非您是像他们一样的服务器网络的所有者，首先他们相较于您自己的解决方案的一个巨大优势是他们使用分布式全球网络系统将您的图像副本更贴近您的用户。 随着趋势的变化，CDN也可以更容易在未来验证您的图像加载策略 - 自行完成此操作需要维护，跟踪浏览器对新兴格式的支持以及跟随图像压缩社区。

其次，每个服务都有分层定价计划，与高容量溢价计划相比，Cloudinary 提供[免费](http://cloudinary.com/pricing)的定价，imgix 以低廉的价格为其标准级别定价。Imgix 提供免费[试用版](https://www.imgix.com/pricing)，并享有服务积分，因此几乎相当于免费级别。

第三，API 访问由两个服务提供。开发人员可以以编程方式访问 CDN 并自动处理。还提供客户端库、框架插件和 API 文档，某些功能仅限于较高付费级别。

Third, API access is provided by both services. Developers can access the CDN programmatically and automate their processing. Client libraries, framework plugins, and API documentation are also available, with some features restricted to higher paid levels.

**让我们开始图像处理**

现在，让我们将讨论局限于静态图像。 Cloudinary和Imgix都提供了一系列图像处理方法，并且在标准和免费计划中都支持压缩，调整大小，裁剪和缩略图创建等主要功能。



<img
        class="lazyload"
        data-src="images/book-images/Modern-Image36-large.jpg"
        alt="cloudinary media library"
         />





> Cloudinary Media Library：默认情况下，Cloudinary对[非渐进式JPEG](http://cloudinary.com/blog/progressive_jpegs_and_green_martians)进行编码。 要选择生成它们，请选中“更多选项”中的“渐进式”选项或传递“fl_progressive”标记。



Cloudinary列出了[七个常用的图像转换类别](http://cloudinary.com/documentation/image_transformations)，其中共有48个子类别。 Imgix宣传支持[100多个图像处理操作](https://docs.imgix.com/apis/url?_ga=2.52377449.1538976134.1501179780-2118608066.1501179780)s。



**默认情况会发生什么？**

Cloudinary默认执行以下优化：

- [使用MozJPEG对JPEG进行编码](https://twitter.com/etportis/status/891529495336722432)（默认选择Guetzli）
- 从变换后的图像文件中剥离所有关联的元数据（原始图像保持不变）。 要覆盖此行为并提供其元数据完整的转换后的图像，请添加keep_iptc标志。

*   可以生成具有自动质量的WebP，GIF，JPEG和JPEG-XR格式。 要覆盖默认调整，请在转换中设置quality参数。
*   在生成PNG，JPEG或GIF格式的图像时，运行[优化](http://cloudinary.com/documentation/image_optimization#default_optimizations) 算法以最小化文件大小，同时对视觉质量的影响最小。



Imgix不像Cloudinary一样有默认优化选项。 它具有可设置的默认图像质量。 对于imgix，自动参数可帮助您在整个图像目录中自动化基线优化级别。

目前，它有[四种不同的方法](https://docs.imgix.com/apis/url/auto)：

*   压缩
 - 视觉增强
 - 文件格式转换
 - 去除红眼



Imgix支持以下图像格式：JPEG，JPEG2000，PNG，GIF，动画GIF，TIFF，BMP，ICNS，ICO，PDF，PCT，PSD，AI

Cloudinary支持以下图像格式：JPEG，JPEG 2000，JPEG XR，PNG，GIF，动画GIF，WebP，动画WebP，BMP，TIFF，ICO，PDF，EPS，PSD，SVG，AI，DjVu，FLIF，TARGA。



**性能如何？**

CDN分发性能主要是关于[延迟](https://docs.google.com/a/chromium.org/viewer?a=v&pid=sites&srcid=Y2hyb21pdW0ub3JnfGRldnxneDoxMzcyOWI1N2I4YzI3NzE2) 和速度。

对于完全未缓存的图像，延迟总是有所增加。 但是，一旦图像被缓存并在网络服务器之间分布，全局CDN可以找到最短的用户节点，加上正确处理的图像的字节节省，与图像处理不当的单机节点试图覆盖整个全球范围相比，几乎总能缓解延迟问题。

（译者注：hop 是网络请求查询路由表的下一个就近节点，俗称下一跳，此处翻译为节点比较合适。）

这两种服务商都使用快速和分布广泛的CDN。 此配置可减少延迟并提高下载速度。 下载速度会影响页面加载时间，这是用户体验和转换的最重要指标之一。



**那么他们如何比较？**

Cloudinary拥有[160,000名客户](http://cloudinary.com/customers)，包括Netflix，eBay和Dropbox。 Imgix没有报告它有多少客户，但它比Cloudinary小一些。 即便如此，imgix的用户基础包括重量级图像用户，如Kickstarter，Exposure，unsplash和Eventbrite。

在图像处理中存在许多不受控制的变量，两个服务之间的点对点的性能比较是困难的。 这在很大程度上取决于您需要处理多少图像 - 这需要不同的时间 - 以及最终输出所需的大小和分辨率，这会影响速度和下载时间。 成本最终可能是最重要的因素。

在图像处理中存在许多不受控制的变量，两个服务之间的头对头性能比较是困难的。 这在很大程度上取决于您需要处理多少图像 - 这需要不同的时间 - 以及最终输出所需的大小和分辨率，这会影响速度和下载时间。 成本最终可能是最重要的因素。

CDN需要花钱。 拥有大量流量的图像繁重的网站每月可能需要花费数百美元的CDN费用。 为了充分利用这些服务，需要一定程度的必备知识和编程技能。 如果你没有做任何太花哨的事情，你可能不会有任何麻烦。

但是，如果您不习惯使用图像处理工具或API，那么您正在考虑一些学习途径。 为了容纳CDN服务器位置，您需要更改本地链接中的某些URL。 请尽职做好正确的调查。:)



**结论**

如果您目前正在提供自己的图像或有这方面的计划，也许您也应该考虑CDN。



## <a id="caching-image-assets" href="#caching-image-assets">缓存图像资源</a>

资源可以使用[HTTP缓存标头](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching#cache-control)指定缓存策略。 具体来说，“Cache-Control”可以定义谁可以缓存响应以及响应时间

您向用户提供的大多数图像都是静态资产，将来[不会发生变化](http://kean.github.io/post/image-caching)。 这类资产的最佳缓存策略是主动缓存。

设置HTTP缓存标头时，将Cache-Control设置为max-age为一年（例如`Cache-Control:public; max-age = 31536000`）。 这种类型的较为激进的缓存策略适用于大多数类型的图像，特别是那些像头像和图像标题一样长久不变的图像。

**注：**如果您使用PHP提供图像，则由于默认的[session_cache_limiter](http://php.net/manual/en/function.session-cache-limiter.php)设置，它可以破坏缓存。 这可能是图像缓存的灾难，您可能希望通过设置session_cache_limiter（'public'）来[解决此问题](https://stackoverflow.com/a/3905468)，该会话将设置为`public, max-age =`。 禁用和设置自定义缓存控制标头也可以。



## <a id="preload-critical-image-assets" href="#preload-critical-image-assets">预加载关键图像资产</a>

可以使用[`<link rel = preload>`](https://www.w3.org/TR/preload/)预加载关键图像资源。

`<link rel=preload>` is a declarative fetch, allowing you to force the browser to make a request for a resource without blocking the document’s `onload` event. It enables increasing the priority of requests for resources that might otherwise not be discovered until later in the document parsing process. 

`<link rel = preload>`是一个声明性提取，允许您强制浏览器发出资源请求而不阻塞文档的`onload`事件。 它可以提高资源请求的优先级，否则在文档解析过程的后期才能发现这些资源。

可以通过指定`image`的`as`值来预加载图像：

```html
<link rel="preload" as="image" href="logo.jpg"/>
```

`<img>`，`<picture>`，`srcset`和SVG的图像资源都可以利用这种方式进行优化。

**注：**Chrome和基于Blink的浏览器（如Opera，[Safari预览版](https://developer.apple.com/safari/technology-preview/release-notes/)） [支持](http://caniuse.com/#search=preload)`<link rel =“preload”>`，并且Firefox也已经[实现](https://bugzilla.mozilla.org/show_bug.cgi?id=1222633)了该方法。

[Philips](https://www.usa.philips.com/)，[Flipkart](https://www.flipkart.com/) 和[Xerox](https://www.xerox.com/) 等网站使用`<link rel = preload>`来预加载其主要LOGO（通常在文档的早期使用）。 [Kayak](https://kayak.com/) 还使用预加载来确保尽快加载其标题页的主页横幅。

<img
        class="lazyload"
        data-src="images/book-images/preload-philips-large.jpg"
        alt="Philips use link rel=preload to preload their logo image"
         />



**什么是链接预加载头？**

可以使用HTML标签或[HTTP链接标头](https://www.w3.org/wiki/LinkHeader)指定预加载链接。 在任何一种情况下，预加载链接都会指示浏览器开始将资源加载到内存缓存中，这表明页面期望高可信度地使用该资源，并且不希望等待预加载扫描程序或解析器发现它。

图像的链接预加载头看起来类似于：

```
Link: <https://example.com/logo-hires.jpg>; rel=preload; as=image
```

当英国“金融时报”向其网站引入了链接预加载标题时，他们将显示其标题图像所花费的时间[缩短了1秒](https://twitter.com/wheresrhys/status/843252599902167040)：


<img
        class="lazyload"
        data-src="images/book-images/preload-financial-times-large.jpg"
        alt="The FT using preload. Displayed are the WebPageTest before and after traces showing improvements."
         />



> 下方: 使用 `<link rel=preload>`, 上方: 并未。在WebPageTest 上Moto G4 在3G情况下的[之前](https://www.webpagetest.org/result/170319_Z2_GFR/)和[之后](https://www.webpagetest.org/result/170319_R8_G4Q/)的对比。



同样，维基百科通过全方位[研究](https://phabricator.wikimedia.org/phame/post/view/19/improving_time-to-logo_performance_with_preload_links/)其他案例中的链接预加载标头，显著提升了实时加载Logo的性能。



**使用此优化时应考虑哪些注意事项？**

请确保将图像资源预加载是非常值得做，因为如果它们对您的用户体验不是很重要，那么页面上可能还有其他内容值得集中精力进行更早的加载。 通过优先处理图像请求，您最终可能会将其他资源推到后续队列中。

在没有广泛的浏览器支持（例如WebP）的情况下，避免使用`rel = preload`来预加载某种图像格式是很重要的。 避免将它用于`srcset`中定义的响应式图像也是很重要的，因为其检索到的图像源可能根据设备条件而变化。

要了解有关预加载的更多信息，请参阅文章[《在Chrome中预加载，预取和优先级》](https://medium.com/reloading/preload-prefetch-and-priorities-in-chrome-776165961bbf)和[《预加载：它有什么用？》](https://www.smashingmagazine.com/2016/02/preload-what-is-it-good-for/)



## <a id="performance-budgets" href="#performance-budgets">图像的Web性能指标</a>

性能指标是团队尝试不超过的网页性能的“指标”。 例如，“任何页面上的图像不会超过200KB”或“用户体验必须在3秒内可用”。 如果没有达到预期，请探究原因以及如何重新达到目标。

与利益相关者讨论绩效时，指标是一个有用的参考框架。 如果设计或业务决策可能会影响网站性能，请参考性能指标。 当更改可能损害站点的用户体验时，这是推倒更改或重新思考更改的一个参考依据。

（译者注：原文中使用budget，直译为预算，私以为翻译为指标更通顺一些。）

我发现在监控自动化时，团队在性能指标方面取得了巨大的成功。 自动化可以标记何时超出预期指标，而不是在预算复盘时手动检查网络资源加载瀑布流。 对性能预算跟踪有用的两个此类服务是[Calibre](https://calibreapp.com/docs/metrics/budgets)和[SpeedCurve](https://speedcurve.com/blog/tag/performance-budgets/)。

（译者注：network waterfall译为了“网络资源加载瀑布流”，也就是在Chrome的开发者工具的Network选项里显示数据流。）

一旦定义了图像大小的性能预算，SpeedCurve就会开始监控并在超出预算时提醒您：



<img
        class="lazyload small"
        data-src="images/book-images/F2BCD61B-85C5-4E82-88CF-9E39CB75C9C0-large.jpg"
        alt="SpeedCurve image size monitoring."
         />



Calibre提供了类似的功能，支持为您定位的每个设备级别设置指标。 这非常有用，因为您通过WiFi在个人PC桌面上调整图片大小的指标可能会因移动设备的指标而有很大差异。

<img
        class="lazyload small"
        data-src="images/book-images/budgets-large.jpg"
        alt="Calibre supports budgets for image sizes."
         />



## <a id="closing-recommendations" href="#closing-recommendations">最后的建议</a>

最终，选择图像优化策略将归结为您向用户提供的图像类型，以及您决定的一组合理的评估标准。 它可能正在使用SSIM或Butteraugli或其他，如果它是一组足够小的图像，那这么做影响到人们对图像的感知。

（译者注：对于小图像再进行优化实际上可能会影响到图片的显示效果。）

**以下是我最后的建议**

如果您**不能**基于浏览器的支持情况有条件的提供对应的格式：


* Guetzli + MozJPEG的jpegtran对于JPEG图像质量大于90是一组不错的优化器。
    * 对于网络图像`q = 90`是有些浪费，你可以使用`q = 80`，并且在2×显示器上即使用`q = 50`也可以。 由于Guetzli没有那么低设置，因此对于网络图像你可以使用MozJPEG。
    * Kornel Lesi&#x144;ski  最近改进了mozjpeg cjpeg命令，添加了轻量级的sRGB配置文件，以帮助Chrome在宽色域显示器上显示自然色彩
* PNG方面 pngquant + advpng具有非常好的速度/压缩比
* 如果你**可以**提供有条件的服务：（使用`<picture>`, the [Accept header](https://www.igvita.com/2013/05/01/deploying-webp-via-accept-content-negotiation/) 或[Picturefill](https://scottjehl.github.io/picturefill/)）：
    * 将WebP提供给支持它的浏览器
        * 从原始的100％质量图像创建WebP图像。 否则你会给那些支持它的浏览器带来更糟糕的JPEG扭曲*以及* WebP扭曲！ 如果使用WebP对未压缩的源图像进行压缩，那么它将具有不太明显的WebP扭曲，并且也可以更好地进行压缩。
        * WebP团队使用`-m 4 -q 75`的默认设置通常适用于大多数情况下优化*速度/比率*的情况。
        * WebP还有一个无损（-m 6 -q 100`）的特殊模式，它可以通过浏览所有参数组合将文件缩小到最小尺寸。 它的速度要慢一个数量级，但静态资产却值得。
    *   作为备选方案，将Guetzli / MozJPEG压缩源提供给其他浏览器

开心快乐的进行压缩吧！

（译者注：哈撒剋！面对疾风吧！）

**Note：** 有关优化图像更实用的指导，我强烈推荐Jeremy Wagner的《[Web 高性能实战](https://www.manning.com/books/web-performance-in-action)》。《[高性能图像](http://shop.oreilly.com/product/0636920039730.do) 》也包含很多关于这个主题的优秀、精彩的建议。



## <a id="trivia" href="#trivia">后记</a>

* [JPEG XT](https://jpeg.org/jpegxt/)定义了1992 JPEG规范的扩展。 对于旧JPEG上具有像素完美渲染的扩展，规范必须澄清旧的1992规范，并选择[libjpeg-turbo](https://libjpeg-turbo.org/)作为其参考实现（基于 受欢迎程度）。
* [PIK](https://github.com/google/pik)是值得关注的新图像编解码器。 它与JPEG兼容，具有更高效的色彩空间，并利用Guetzli中的类似优势。 它的解码速度是JPEG的2/3，比libjpeg节省了54％的文件。 解码和压缩比Guetzli-ified JPEG更快。 关于现代图像代码的心理视觉相似性的[研究](https://encode.ru/threads/2814-Psychovisual-analysis-on-modern-lossy-image-codecs)显示PIK不到替代品的一半。 不幸的是，目前编解码器和编码仍处于初期阶段（2017年8月）非常缓慢。
* [ImageMagick](https://www.imagemagick.org/script/index.php)通常被推荐用于图像优化。 这篇文章认为它是一个很好的工具，但它的输出通常需要更多的优化调整，其他工具也可以提供更好的输出。 我们建议尝试[libvips](https://github.com/jcupitt/libvips)，但它的优先级更低，需要更多技术技能才能使用。 ImageMagick还可以在社区上有[注意](https://imagetragick.com/#moreinfo)您可能想要了解的安全漏洞。
* Blink（Chrome使用的渲染引擎）从主线程解码图像。 将解码工作移动到合成器线程可以释放主线程以处理其他任务。 我们称之为延迟解码。 通过延迟解码，在用于向显示器呈现帧上解码工作仍然是关键任务，因此它仍然可以导致动画抖动。 [`img.decode()`](https://html.spec.whatwg.org/multipage/embedded-content.html#dom-img-decode)API应该有助于解决该问题。



本书的内容是在Creative Commons 下的 [Attribution-NonCommercial-NoDerivs 2.0 Generic (CC BY-NC-ND 2.0)](https://creativecommons.org/licenses/by-nc-nd/2.0/)许可下授权的，代码样本是在[Apache 2.0 License](http://www.apache.org/licenses/LICENSE-2.0)。谷歌版权所有，2018年。