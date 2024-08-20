# python初步网页操作

在网页操作中，目前我使用的是requests、selenium加上BeautifulSoup进行网页操作

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

