# 源码获取

请下载并解压 [FPGA_D42_SW1_10280301.rar](FPGA_D42_SW1_10280301.rar)。下面的工程路径均指解压后的目录。源码、测试、窗口和码流均在压缩包内。

# FPGA D42 SW1 一键起摆完整源码（10280301）

本包包含可独立打开、构建的高云工程和全部启用的 RTL、约束、配套窗口源码/依赖、数字测试及已生成码流。固件版本 10280301，目标芯片 GW2A-LV55PG484C8/I7，顶层 auto_balance_eba_top。

上电后采用存储的竖直零点 ADC12=3127。摆臂放在期望原点，摆杆自然下垂并松手静止，待测量有效后按一次实体 SW1 自动起摆，无需手扶标定。窗口保留“竖直标定”提示且固定显示绿色。实体 SW2 或窗口 STOP 停机，松开后不会自行重启；重新按 SW1 才启动。SW3 停机时恢复同一存储零点。

D42 平衡参数、位置积分、滤波、捕获/失衡恢复、原点、保护与 PWM 保留。窗口定点采用平滑轨迹，45°/s、40°/s²、160°/s³；正弦 ±60°、8 秒周期、6 秒渐入/渐出、总长 36 秒，五点定位已删除。存储零点沿用本机构最近测试值，不会在每次上电重新辨识。

## 文件

- project.gprj、top.v、rtl/ 及根目录其他 .v：完整 FPGA 工程和源文件。
- clock.sdc、j280.cst：时序和引脚约束；build.tcl、build.ps1：构建脚本。
- bitstream/auto_balance_eba_10280301.fs：已通过原 19 ns 时序检查的码流。
- tools/、raw_decoder.py、launch_window.ps1、打开拓展窗口.cmd：窗口及全部自带串口依赖。
- verification/：15 个 RTL 回归的 testbench、输入向量、资格摘要。
- verify_project.py、SOURCE_MANIFEST.json：独立校验/仿真脚本和逐文件 SHA256 清单。
- THIRD_PARTY.md、tools/_vendor/PYSERIAL_LICENSE.txt：第三方说明与许可证。

## 构建与使用

使用 Gowin IDE 打开本目录 project.gprj。已验证的工具版本为 Gowin V1.9.12.03 x64；在 IDE 中执行 Synthesize、Place & Route、Generate Bitstream。也可以在 PowerShell 执行 .\build.ps1 -GowinShell "本机高云安装目录\IDE\bin\gw_sh.exe"。脚本不依赖原开发电脑的磁盘路径。构建结果在 impl/pnr，发布码流位于 bitstream。

窗口需要 Python 3.12 或更新版本并安装 Tcl/Tk。双击“打开拓展窗口.cmd”，或运行 python -X utf8 tools/eba_window.py。pySerial 已自带，无需联网安装。默认连接只接收数据，不启动电机；启动使用实体 SW1。串口参数 921600，选择板上实际串口。

离线检查（均不打开串口）：

```powershell
python -X utf8 verify_project.py
python -X utf8 verify_project.py --elaborate --host-tests
python -X utf8 verify_project.py --rtl-regression --host-tests
```

--elaborate 与 --rtl-regression 需要 Icarus Verilog 的 iverilog/vvp 在 PATH；--host-tests 运行窗口与轨迹的 73 项测试。RTL 测试在临时目录运行，不需要任何历史版本、原始测试日志或开发者目录。tools/analyze_hold_jitter.py 是可选离线分析工具，单独需要 numpy，构建和窗口无需 numpy。

数字回归 17 组通过（15 项 RTL、73 项主机测试、真实 RTL UART 解码）；启动面板 100 项检查通过。原 19 ns 时序约束下 Setup/Hold 违例均为 0，最差 Setup 裕量 +1.407 ns。详见 verification/QUALIFICATION.json。数字/实现检查不等于已在本次实物中完成一键起摆验收。
