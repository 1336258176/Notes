1. 生成编译数据库
```bash
colcon build --cmake-args -DCMAKE_EXPORT_COMPILE_COMMANDS=ON -G Ninja
```
构建完成后，请确保 `dev_ws\build` 包含 `compile_commands.json`
2. 用 CLion 作为项目打开 `compile_commands.json`
3. 更改项目根目录
默认情况下，CLion 将包含 `compile_commands.json` 文件的目录视为项目根目录。在我们的例子中，它是 `build` 目录。在项目树中，实际源文件被标记为外部，为了获得正确的项目结构，我们需要将项目根设置为实际的工作空间目录。
依次点击 **工具 |编译数据库|从主菜单更改项目根目录**，然后选择工作区目录即可。（默认情况下，CLion 不会在 `compile_command.json` 发生更改时自动重新加载项目，VCS 等外部事件的情况除外更新。您可以在 **设置|构建|执行|部署|构建工具** 中更改此行为。）

- https://www.jetbrains.com/help/clion/2023.3/ros2-tutorial.html#build-pkg
- https://www.jetbrains.com/help/clion/2023.3/compilation-database.html#code-insight-json
- https://www.jetbrains.com/help/clion/compilation-database.html#ninja

Bus 003 Device 008: ID 0483:5740 QinHeng Electronics USB Single Serial