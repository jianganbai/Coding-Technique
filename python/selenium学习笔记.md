# selenium学习笔记

- 用代码模拟浏览器点击、输入密码等
- 可用于爬虫、网站测试等

- 安装：以chrome为例
  - 到selenium下载适配chrome版本的chromedriver
  - 在conda环境中安装selenium

## 初始化

```python
from selenium import webdriver

option = webdriver.ChromeOptions()
option.add_argument('--headless')  # 不展示浏览器访问过程

browser = webdriver.Chrome(executable_path=驱动位置, options=option)
broser.get(url)  # 打开浏览器，访问url
```

### options

- 设置编码格式：`options.add_argument('lang=zh_CN.UTF-8')`，utf-8格式

- 模拟移动设备

  - ```python
    options.add_argument('user-agent="填入移动设备的user-agent信息"')
    ```

  - user-agent查询地址：http://www.fynas.com/ua，对应手机型号、系统、浏览器

- 禁止图片加载

  - ```python
    prefs = {"profile.managed_default_content_setting.images": 2}
    chrome_options.add_experimental_options("prefs", prefs)
    ```

## 定位元素

- ```python
  browser.find_element_by_css_selector(CSS路径)
  browser.find_element(类型，值)  # selenium 4.0之后
  ```
  
  - F12，找到该元素的代码，邮件复制，选择复制selector，即为CSS路径
  - 成功则返回True，失败则抛出异常
  
- ```python
  browser.find_elements_by_class_name(所属类名)
  ```

  - `find_elements`：寻找所有同类名的对象

- **返回对象，可对该元素直接操作**

  - `.text`：获得文本
  - `.get_attribute(属性名)`：获得元素的属性值
  - `.click()`：点击该元素
  - `.send_keys(str)`：填入字符串
  - `.clear()`：清空文字

## 等待

### 强制等待

- `time.sleep()`

### 隐式等待

- 隐式等待：若元素存在，则自动向下执行，超出则抛出异常

- `browser.implicitly_wait(5)`：写在刚创建browser后，之后所有操作都隐式等待5s

### 显式等待

- 显式等待：能自定义条件，满足条件才向下执行

- ```python
  from selenium.webdriver.support.wait import WebDriverWait
  from selenium.webdriver.support import expected_conditions as EC
  from selenium.webdriver.common.by import By
  
  WebDriverWait(driver, timeout, poll_frequency=0.5, ignored_exceptions=None)
  # timeout: 等待上限
  # poll_frequency: 检查条件的周期
  # ignored_exceptions: 超时后抛出的异常信息
  
  # 等id=TANGRAM的元素加载出来
  WebDriverWait(driver, 10, 0.8).until(EC.presence_of_element_located(By.ID, 'TANGRAM'))
  ```

## 执行JS命令

- `browser.excecute_script(JavaScript脚本代码字符串)`

### JS命令例

- `window.scrollTo(x坐标, y坐标)`：滚动页面，x是左右坐标，y是上下坐标
  - `window.scrollTo(0,document.body.scrollHeight)`：滚动到最下方

## 反反爬虫机制

- 网站通过JS脚本，检测网页的JS值，来鉴别是真人还是selenium
- 要么在执行网站的JS之前，先用CDP执行JS修改这些值
- 要么修改chromedriver，修改相应JS变量名

### 修改webdriver标签

- 使用selenium，`window.navigator.webdriver`会为true，正常浏览为false

  - 浏览器给每个窗口一个window属性来记录用户信息
  - 使用F12，在console中输入，可查询

- ```python
  # 新加入如下
  option.add_argument('--disable-blink-features=AutomationControlled')  # 去除特征值
  self.browser.execute_cdp_cmd("Page.addScriptToEvaluateOnNewDocument", {"source": """Object.defineProperty(navigator, 'webdriver', {get: () => undefined})"""})
  ```

## 其它

- 切换网页：`browser.switch_to.frame(网页编号)`