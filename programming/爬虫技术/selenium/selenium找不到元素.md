### 1.元素遮挡
```python
driver.find_element_by_css_selector("div.loginForm>input#loginBtn").send_keys(Keys.ENTER)
```
### 2.元素动态加载
### 3.元素在iframe中
```python
driver.switch_to_frame("frameName")  # 根据框架名来切换

driver.switch_to_frame("frameName.0.child")  # 子框架

driver.switch_to_default_content()  # 返回
```
### 4.元素未加载完成
### 5.滚动条
```python
A．让滚动条定位到指定元素位置

menu = driver.find_element_by_css_selector(".nav")

hidden_submenu = driver.find_element_by_css_selector(".nav #submenu1")

ActionChains(driver).move_to_element(menu).click(hidden_submenu).perform()

B.尝试下拉一段滚动条，让按钮能看到

js = "window.scrollTo(100,450)"

driver.execute_script(js)

driver.find_element_by_css_selector("div.loginForm>input#loginBtn").click()
```
### 6.反爬

### 7.其他
报错
```shell
selenium.common.exceptions.StaleElementReferenceException: Message: Element not found in the cache
解决
在网上搜了很多解决方案都不得法，最后在重新查找元素前加入如下代码妥妥解决

time.sleep(1)
如论是刷新重新加载还是翻页，再遇到这样的问题的时候，先等一等在获取，页面就不是之前的了

time.sleep(1)
line_list = self._driver.find_element_by_xpath("//div[@id='content_left'][1]").find_elements_by_xpath("//div[@class='result c-container ']")
```