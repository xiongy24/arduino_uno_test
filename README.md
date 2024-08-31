## 在 Ubuntu 环境下使用 VSCode 和 PlatformIO 下载程序到 Arduino Uno

1. 安装 VSCode
   - 访问 [VSCode 官网](https://code.visualstudio.com/) 下载 .deb 包
   - 使用以下命令安装：
     ```
     sudo dpkg -i <下载的文件名>.deb
     sudo apt-get install -f
     ```

2. 安装 PlatformIO 扩展
   - 在 VSCode 中，转到扩展市场（Ctrl+Shift+X）
   - 搜索 "PlatformIO IDE"
   - 点击 "安装"

3. 创建 PlatformIO 项目
   - 在 VSCode 中，点击 PlatformIO 图标（通常在左侧栏）
   - 选择 "New Project"
   - 选择 Board: "Arduino Uno"
   - 选择 Framework: "Arduino"
   - 选择项目位置并命名

4. 编写代码
   - 在 `src/main.cpp` 中编写或粘贴您的 Arduino 代码
     ```
     #include <Arduino.h>

     void printMenu() {
         Serial.println("\n--- Arduino Menu ---");
         Serial.println("1. Say Hello");
         Serial.println("2. Get Arduino Uptime");
         Serial.println("3. Blink LED");
         Serial.println("Enter your choice:");
     }

     void setup() {
         Serial.begin(9600);
         while (!Serial) {
             ; // Wait for serial port to connect (needed for native USB port only)
         }
         Serial.println("Arduino ready for communication!");
         pinMode(LED_BUILTIN, OUTPUT);
         printMenu();
     }

     void loop() {
         if (Serial.available() > 0) {
             char choice = Serial.read();
             
             // Clear the serial buffer
             while (Serial.available() > 0) {
                 Serial.read();
             }
             
             switch (choice) {
                 case '1':
                     Serial.println("Hello from Arduino!");
                     break;
                 case '2':
                     Serial.print("Arduino uptime: ");
                     Serial.print(millis() / 1000);
                     Serial.println(" seconds");
                     break;
                 case '3':
                     Serial.println("Blinking LED 3 times...");
                     for (int i = 0; i < 3; i++) {
                         digitalWrite(LED_BUILTIN, HIGH);
                         delay(500);
                         digitalWrite(LED_BUILTIN, LOW);
                         delay(500);
                     }
                     Serial.println("Blinking complete!");
                     break;
                 default:
                     Serial.println("Invalid choice. Please try again.");
             }
             
             printMenu();
         }
     }
     ```

5. 连接 Arduino Uno
   - 使用 USB 线将 Arduino Uno 连接到电脑

6. 编译和上传
   - 点击 VSCode 底部状态栏的 "PlatformIO: Build" 图标编译代码
   - 点击 "PlatformIO: Upload" 图标将程序上传到 Arduino Uno

7. 监视串口输出
   - 点击 "PlatformIO: Serial Monitor" 图标打开串口监视器
   - 这时在串口监视器中输入数字，就可以选择相应的功能。

   注：如果遇到权限问题，可能需要将用户添加到 `dialout` 组：
   ```
   sudo usermod -a -G dialout $USER
   ```
   添加后需要注销并重新登录才能生效。

## 故障排除

### 串口监视器错误

如果在点击 "PlatformIO: Serial Monitor" 时出现类似以下的错误：
![错误截图](4d4a4b89dedc6a9856a0f0fa0785967.png)

这是因为 PlatformIO 在尝试运行测试，但是找不到测试文件。解决方法如下：

1. 确保您不是在运行测试命令。串口监视器应该使用 "Monitor" 命令，而不是 "Test" 命令。

2. 如果问题仍然存在，尝试以下步骤：
   - 在项目根目录创建一个 `test` 文件夹（如果不存在）
   - 在 `test` 文件夹中创建一个空的测试文件，例如 `test_main.cpp`
   - 在 `test_main.cpp` 中添加以下内容：
     ```
     #include <unity.h>

     void setUp(void) {
         // set stuff up here
     }

     void tearDown(void) {
         // clean stuff up here
     }

     void test_function_calculator_addition(void) {
         TEST_ASSERT_EQUAL(32, 25 + 7);
     }

     int main(int argc, char **argv) {
         UNITY_BEGIN();
         RUN_TEST(test_function_calculator_addition);
         UNITY_END();

         return 0;
     }
     ```

3. 尝试使用命令行运行串口监视器：
   - 打开终端
   - 导航到项目目录
   - 运行命令：`pio device monitor`

4. 检查 `platformio.ini` 文件，确保正确配置了串口监视器：
   ```
   [env:uno]
   platform = atmelavr
   board = uno
   framework = arduino
   monitor_speed = 9600
   ```

如果问题仍然存在，可以尝试重新安装 PlatformIO 或查看 PlatformIO 的官方文档以获取更多帮助。

