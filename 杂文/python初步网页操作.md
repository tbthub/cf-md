# python网页处理学习记录

## requests:

#### 主要作用

- 发送HTTP请求（如GET，POST）
- 处理相应数据
- 设置请求头

#### 示例代码

```python
import requests
get_url = 'https://get.com'
get_response = requests.get(get_url)
print(get_response.text)

headers = {'Content-Type': 'application/x-www-form-urlencoded'}
post_url = 'https://post.com'
post_response = requests.post(post_url,headers=headers)
...
```



## selenium

#### 主要作用

- 模拟浏览器行为，可以操作浏览器进行自动化测试或爬取动态加载的数据。
  支持多种浏览器（Chrome, Firefox, Safari 等）。

注意：`chrome与Chromedriver版本一致`

[chrome linux官网](https://www.google.com/chrome/?platform=linux)

[chrome deb](https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb)

[chromedriver下载地址(127以后)](https://googlechromelabs.github.io/chrome-for-testing/)

[chromediiver版本选择指引](https://developer.chrome.com/docs/chromedriver/downloads/version-selection?hl=zh-cn)

#### 示例代码

```python
from selenium import webdriver

service = Service('/usr/bin/chromedriver')  # 修改为你的Chromedriver的实际路径
# 创建浏览器实例
driver = webdriver.Chrome(service=service)
driver.get('https://example.com')
# 异步
wait = WebDriverWait(driver, 10)
try:
    hello_world = driver.find_element(By.ID, 'Hello World')
    #元素加载可见后（这里是根据ID寻找元素）
    input_element = wait.until(EC.visibility_of_element_located((By.ID, 'input_ID')))
    input_element.click()
    #清除输入框的值
    input_element.clear()
    #设置值
    input_element.send_keys('your content')
    #设置值后带有一个回车，可用于网页要根据输入值后要重新显示新数据
    input_element.send_keys('your content', Keys.ENTER)
	
    #等到元素可以点击后才继续下去
    btn = wait.until(
        EC.element_to_be_clickable(
            (By.XPATH, 'element_XPATH'))
    )
    btn.click()
    
    #有些时候有些元素有遮罩，导致click()无法直接点击，这个时候可以使用
    #这里假设element无法点击
    #from selenium.webdriver.common.action_chains import ActionChains
    
    #actions = ActionChains(driver)
    #actions.move_to_element(element).click().perform()
driver.quit()
```

以下适用于服务器黑乎乎控制台模式

```
# Chrome的无头模式配置
chrome_options = Options()
chrome_options.add_argument('--headless')
chrome_options.add_argument('--disable-gpu')
chrome_options.add_argument('--no-sandbox')
chrome_options.add_argument('--disable-dev-shm-usage')

service = Service('/usr/bin/chromedriver')  # 修改为你的Chromedriver的实际路径
driver = webdriver.Chrome(service=service, options=chrome_options)
```



## BeautifulSoup

#### 主要作用

- 解析 HTML 或 XML 文档。提供方法来搜索、提取所需的数据。

#### 示例代码

```python
from bs4 import BeautifulSoup
import requests
get_url = 'https://get.com'
get_response = requests.get(get_url)

#放进网页html
soup = BeautifulSoup(get_response.text, 'html.parser')

# 获取所有的链接
links = soup.find_all('a')
forpy link in links:
    print(link.get('href'))
...
```



## 如何傻瓜式的写一个

理论上一个requests啥都能处理，不过需要在浏览器开发者调试里面一个一个看网页的网络请求。。。而且碰上一些不不知名的传输值，还得去网页里面畅游（虽然浏览器网页可以直接看源码，不过ψ(｀∇´)ψ），碰上故意加密的。。。慢慢去js找吧）

既然这样，目前感觉最简单的，就是selenium+BeautifulSoup(可选)。因为其可以直接模拟用户操作的缘故，我们不必关心下面网页自己的实现，只需要在代码按照我们自己操作的逻辑就可以轻松实现。总结起来就是两步骤：

1. 选元素
2. 操作元素。over!



### 示例：


实现：主要利用python的selenium实现模拟点击，操控浏览器。

功能：广州大学图书管座位预约

需要：chrome和对应的[chrome driver](https://developer.chrome.com/docs/chromedriver/downloads?hl=zh-cn)（**注意版本号要一致！**）

流程：数字广大登录 ->选择书库->选择时间->选择座位->提交

`main.py`

```python
import time
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.action_chains import ActionChains  # 添加这一行
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.common.keys import Keys
from rooms import time_list
from rooms import Rooms

# Chrome的无头模式配置（无头模式需要，不用无头模式可自行注释）
chrome_options = Options()
chrome_options.add_argument('--headless')
chrome_options.add_argument('--disable-gpu')
chrome_options.add_argument('--no-sandbox')
chrome_options.add_argument('--disable-dev-shm-usage')

# 设置ChromeDriver的路径
# service = Service('D:\doc\chromedriver-win64\chromedriver.exe')  # 修改为你的Chromedriver的实际路径
service = Service('/usr/bin/chromedriver')  # 修改为你的Chromedriver的实际路径

initial_url = 'https://libbooking.gzhu.edu.cn/#/'


def reserve(username, password, room_name, seat_number, dateTime, startTime, endTime):
    # 创建浏览器实例
    driver = webdriver.Chrome(service=service, options=chrome_options)	#无头模式，适合在服务器等无图形化界面使用
    # driver = webdriver.Chrome(service=service)	#图形化界面下可以看
    wait = WebDriverWait(driver, 10)
    print("开始预约")

    driver.get(initial_url)
    actions = ActionChains(driver)

    try:
        print("进入登录页面-----------------------------------------------------------------")
        # 等待页面加载完成后,寻找并显示用户名输入框
        un = wait.until(EC.visibility_of_element_located((By.ID, 'un')))
        un.click()
        un.send_keys(username)

        # 寻找并显示密码输入框
        pd = driver.find_element(By.ID, 'pd')
        pd.click()
        pd.send_keys(password)

        # 找到登录按钮并点击
        login_button = driver.find_element(By.ID, 'index_login_btn')
        login_button.click()

        print("进入IC空间-----------------------------------------------------------------")

        # 获取书库
        room_index = Rooms.index(room_name) + 1
        room = wait.until(
            EC.element_to_be_clickable(
                (By.XPATH, f"/html/body/div/div[1]/div[3]/div[2]/div[1]/div/div[1]/div/div[2]/div[{room_index}]"))
        )
        room.click()
        print(1)

        # 选择日期
        date = wait.until(
            EC.element_to_be_clickable(
                (By.XPATH, "/html/body/div[1]/div[1]/div[3]/div[2]/div[4]/div/section/div/input")
            )
        )
        date.clear()
        date.send_keys(dateTime, Keys.ENTER)
        print(2)

        # 选择起始时间
        start_time = wait.until(
            EC.element_to_be_clickable((By.XPATH, "//input[@placeholder='请选择开始时间']"))
        )
        actions.move_to_element(start_time).click().perform()
        time.sleep(0.5)
        s_time_index = time_list.index(startTime) + 1
        td1 = wait.until(
            EC.element_to_be_clickable(
                (By.XPATH, f'/html/body/div[3]/div[1]/div[1]/ul/li[{s_time_index}]')
            )
        )
        td1.click()
        print(3)

        # 选择结束时间
        end_time = wait.until(
            EC.element_to_be_clickable((By.XPATH, "//input[@placeholder='请选择结束时间']"))
        )
        actions.move_to_element(end_time).click().perform()
        time.sleep(0.5)
        e_time_index = time_list.index(endTime) - s_time_index - 10
        td2 = wait.until(
            EC.element_to_be_clickable(
                (By.XPATH, f'/html/body/div[4]/div[1]/div[1]/ul/li[{e_time_index}]')
            )
        )
        td2.click()
        print(4)

        # 获取座位
        seat = wait.until(
            EC.element_to_be_clickable(
                (By.XPATH, f"/html/body/div[1]/div[1]/div[3]/div[2]/div[4]/div/div[2]/div[1]/div[{seat_number}]"))
        )
        actions.move_to_element(seat).click().perform()
        print(5)

        # 提交
        submit = wait.until(
            EC.element_to_be_clickable(
                (
                    By.XPATH,
                    f"/html/body/div[1]/div[1]/div[3]/div[2]/div[4]/div/div[3]/div[1]/div/div[3]/span/button[1]"))
        )
        submit.click()
        print(f"预约成功 >>>>>>>>>>>>>>>")
        print(room_name, seat_number, dateTime, startTime, endTime)

    except:
        print("预约失败 #################")

    finally:
        # 退出浏览器
        driver.quit()
        return True
```

`rooms.py`

```python
Rooms = ['101自修室', '103休闲阅览区',
'202书库', '203书库', '204书库', '205书库', '206书库', '2楼北面（C区）',
'301书库', '303书库', '3楼现刊阅览室', '3楼南面走廊（A区）', '3楼北面走廊（C区）',
'402书库', '406书库', '417书库', '418书库', '4楼南面走廊（A区）', '4楼北面走廊（C区）',
'501建筑设计书库', '502外文书库', '511室工具、学位论文、廉政书库', '513广州文献资源中心', '514广州文献资源中心', '5楼北面走廊（C区）', '琴房']


time_list =['08:30', '08:35', '08:40', '08:45', '08:50', '08:55', '09:00', '09:05', '09:10', '09:15', '09:20', '09:25',
       '09:30', '09:35', '09:40', '09:45', '09:50', '09:55', '10:00', '10:05', '10:10', '10:15', '10:20', '10:25',
       '10:30', '10:35', '10:40', '10:45', '10:50', '10:55', '11:00', '11:05', '11:10', '11:15', '11:20', '11:25',
       '11:30', '11:35', '11:40', '11:45', '11:50', '11:55', '12:00', '12:05', '12:10', '12:15', '12:20', '12:25',
       '12:30', '12:35', '12:40', '12:45', '12:50', '12:55', '13:00', '13:05', '13:10', '13:15', '13:20', '13:25',
       '13:30', '13:35', '13:40', '13:45', '13:50', '13:55', '14:00', '14:05', '14:10', '14:15', '14:20', '14:25',
       '14:30', '14:35', '14:40', '14:45', '14:50', '14:55', '15:00', '15:05', '15:10', '15:15', '15:20', '15:25',
       '15:30', '15:35', '15:40', '15:45', '15:50', '15:55', '16:00', '16:05', '16:10', '16:15', '16:20', '16:25',
       '16:30', '16:35', '16:40', '16:45', '16:50', '16:55', '17:00', '17:05', '17:10', '17:15', '17:20', '17:25',
       '17:30', '17:35', '17:40', '17:45', '17:50', '17:55', '18:00', '18:05', '18:10', '18:15', '18:20', '18:25',
       '18:30', '18:35', '18:40', '18:45', '18:50', '18:55', '19:00', '19:05', '19:10', '19:15', '19:20', '19:25',
       '19:30', '19:35', '19:40', '19:45', '19:50', '19:55', '20:00', '20:05', '20:10', '20:15', '20:20', '20:25',
       '20:30', '20:35', '20:40', '20:45', '20:50', '20:55', '21:00', '21:05', '21:10', '21:15', '21:20', '21:25',
       '21:30', '21:35', '21:40', '21:45']
```

使用方法：执行`reserve`函数即可

```python
# 学号 密码 书库 座位序号 日期 起始时间 结束时间
#例如 reserve("xxxxx", "yyyyyyyyyy", '203书库', 2, "2024-08-28", "08:30", "09:30")
reserve("xxxxx", "yyyyyyyyyy", '203书库', 2, "2024-08-28", "08:30", "09:30")
```





