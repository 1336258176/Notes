# 说明

- `PnP(Perspective-n-Point)` 是求解3D到2D点相对运动的方法，目的是求解相机坐标系相对世界坐标系的位姿。
- 它描述了已知 $n$ 个3D点的坐标(相对世界坐标系)以及这些点的像素坐标时，如何估计相机的位姿(即求解世界坐标系到相机坐标系的旋转矩阵 $R_{3 \times 3}$ 和平移向量 $\vec{T}_{3 \times 1}$ )。
- 算法：DLT （直接线性变换）、P3P、EPnP、BA（光束法平差）

# DLT

# P3P

# EPnP

# BA

# 代码实现

## OpenCV 函数原型
```cpp
bool cv::solvePnPGeneric(InputArray objectPoints,
						 InputArray imagePoints,
					     InputArray cameraMatrix,
						 InputArray distCoeffs,
					     OutputArrayOfArrays rvecs,
						 OutputArrayOfArrays tvecs,
						 bool useExtrinsicGuess = false.
						 SolvePnpMethod flags = SOLVEPNP_ITERATIVE,
						 InputArray rvec = noArray(),
						 InputArray tvec = noArray(),
						 OutputArray reprojectionError = noArray()
						 )

void cv::Rodrigues(InputArray src,
				   OutputArray dst,
				   OutputArray jacobian = noArray()
				   )
```

# 参考资料