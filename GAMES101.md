# GAMES101

## 基础与一些作业框架有问题的地方

### 定义：

以作业1、2为参考：

```cpp
int main() {
    ...
    Eigen::Vector3f eye_pos = {0, 0, 5};
    ...
    std::vector<Eigen::Vector3f> pos{{2, 0, -2}, {0, 2, -2}, {-2, 0, -2}, {3.5, -1, -5}, {2.5, 1.5, -5}, {-1, 0.5, -5}};
    ...
    r.set_projection(get_projection_matrix(45, 1, 0.1, 50));
    ...
}
```
摄像机位于 `(0,0,5)` 位置，近平面位于 `z = -0.1` 位置，远平面位于 `z = -50` 位置。

### projection矩阵代码：

```cpp
Eigen::Matrix4f get_projection_matrix(float eye_fov, float aspect_ratio, float zNear, float zFar) {
    // TODO: Copy-paste your implementation from the previous assignment.
    Eigen::Matrix4f projection;
    float           fov_radiant = to_radian(eye_fov);
    float           t           = tan(fov_radiant) * zNear;
    float           r           = aspect_ratio * t;

    zNear = -zNear;
    zFar  = -zFar;
    Eigen::Matrix4f m_move;
    m_move << 1, 0, 0, 0,             //
        0, 1, 0, 0,                   //
        0, 0, 1, -(zNear + zFar) / 2, //
        0, 0, 0, 1;

    Eigen::Matrix4f m_scale;
    m_scale << 1 / r, 0, 0, 0,       //
        0, 1 / t, 0, 0,              //
        0, 0, 2 / (zNear - zFar), 0, //
        0, 0, 0, 1;

    Eigen::Matrix4f m2;
    m2 << zNear, 0, 0, 0,                  //
        0, zNear, 0, 0,                    //
        0, 0, zNear + zFar, -zNear * zFar, //
        0, 0, 1, 0;

    projection = m_scale * m_move * m2;
    return projection;
}
```

只需 `zNear` 和 `zFar` 取负数，其它的与上课一样。

### `Rasterizer` 中的 `draw` 函数：

`draw` 函数中有这么一段：

```cpp
for (auto &vert : v) {
    vert.x() = 0.5 * width * (vert.x() + 1.0);
    vert.y() = 0.5 * height * (vert.y() + 1.0);
    vert.z() = vert.z() * f1 + f2;
}
```
目的是将坐标从标准立方体空间变换为屏幕空间。作业指明 `z` 为正数，且 `z` 越大代表离近平面越远，然而以上代码并不能实现此目标，修改后代码如下：

```cpp
for (auto &vert : v) {
        vert.x() = 0.5 * width * (vert.x() + 1.0);
        vert.y() = 0.5 * height * (vert.y() + 1.0);
        vert.z() = abs(vert.z() - 1);
}
```

### `insideTriangle` 函数定义：

原函数定义为：

```cpp
static bool insideTriangle(int x, int y, const Vector3f *_v)
```

此函数用于判断点 `(x ,y)` 是否在三角形内，但是若传入整形则无法达到目标，因此应该为传入浮点数：

```cpp
static bool insideTriangle(float x, float y, const Vector3f *_v)
```