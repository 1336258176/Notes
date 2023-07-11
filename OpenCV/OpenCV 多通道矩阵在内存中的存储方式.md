# Planar 方式（平面方式）

在平面方式下，每个通道的像素值都是连续存储的。也就是说，首先是第一个通道的所有像素值，接着是第二个通道的所有像素值，依此类推。

```
Channel 1: [pixel1, pixel2, pixel3, ...] 
Channel 2: [pixel1, pixel2, pixel3, ...] 
Channel 3: [pixel1, pixel2, pixel3, ...]
```

这种存储方式在处理某些特定的图像操作时可能更高效，例如分别对每个通道进行处理。

# Interleaved 方式（交织方式）

在交织方式下，每个像素的所有通道值是交替存储的。也就是说，首先是第一个像素的所有通道值，接着是第二个像素的所有通道值，依此类推。

```
Pixel 1: [channel1, channel2, channel3, ...] 
Pixel 2: [channel1, channel2, channel3, ...] 
Pixel 3: [channel1, channel2, channel3, ...]
```

在 OpenCV 中，**多通道图像默认使用交织方式存储**，而多通道数组（例如 `cv::Mat` ）则默认使用平面方式存储。这样可以确保与其他库和工具的兼容性。因此有如下代码：

```cpp
cv::Mat img(n, 1, CV_32FC2);
for(int i = 0; i < n; i++)
{
	cv::Point2f p1;
	p1.x = rng.uniform(1,100);
	p1.y = rng.uniform(1.100);
	img.at<cv::Point2f>(i,0) = p1;
}
```