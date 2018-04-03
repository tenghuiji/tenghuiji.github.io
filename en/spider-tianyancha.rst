{
 "cells": [
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# 天眼查爬虫\n",
    "2018-1-21  \n",
    "marking的同事跑过来求帮忙，能不能通过爬虫帮他，根据会议收集表格中，人员名单所填写的公司名称添加一列行业信息。  \n",
    "他目前是手工到天眼输入公司名称，然后再取出觉得匹配的公司，最后在找到行业分类。但是人员明天太多了，做这么枯燥的工作他要疯了:(  \n",
    "我觉得这个应该比较简单，答应他帮他写个爬虫  \n",
    "###### 确定爬取方式\n",
    "selenium是个自动化测试工具，他可以通过浏览器来模拟各种操作实现测试页面功能  \n",
    "BeautifulSoup是个爬虫常用的Dom解析工具  \n",
    "lxml提供了xpath获取方法，强烈推荐通过xpath访问节点，配合Chrome的inspect工具中的 copy xpath可以节省大量的节点获取工作量  \n",
    "###### 安装必要的工具\n",
    "pip install selenium  \n",
    "pip install lxml  \n",
    "pip install beautifulsoup4  \n",
    "下载最新的chromedriver  \n",
    "http://chromedriver.storage.googleapis.com/index.html  \n",
    "我是mac 安装到/usr/local/bin/目录下，然后运行 chromedriver -v查看是否安装好了  \n",
    "###### 常见问题\n",
    "1. 公司名称不存在；在天眼搜里面找不到公司名称就在程序中填写none或者其他的字符说明  \n",
    "2. 相似公司很多个不确定怎么选；这个比较不好处理，现在是取天眼搜中查找到的top 1 记录 我想人选择的话可能会根据与会者所在地配合来查找吧，例如是在北京参加的，那么就有限找北京的同名（名称相似）的公司  \n",
    "3. 天眼里面找到的母公司下面有多个子公司，导致鼠标点击不到连接，必须拉滚动条才能解决问题  \n"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 30,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "from selenium import webdriver\n",
    "from selenium.common.exceptions import TimeoutException\n",
    "from selenium.webdriver.common.by import By\n",
    "from selenium.webdriver.support.ui import WebDriverWait\n",
    "from selenium.webdriver.support import expected_conditions as EC\n",
    "import pandas as pd\n",
    "from bs4 import BeautifulSoup\n",
    "from lxml import etree\n",
    "import time\n",
    "import random"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 31,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "def search(keywords):\n",
    "    # print('正在搜索:{}'.format(keywords))\n",
    "    try:\n",
    "        browser.get('https://www.tianyancha.com/')\n",
    "        # 找到输入框\n",
    "        input = wait.until(\n",
    "            EC.presence_of_element_located(\n",
    "                (By.CSS_SELECTOR,\n",
    "                 '.main-tab-outer .main-tab-body .inputV2 input')))\n",
    "        # input.send_keys(keywords.decode('utf-8'))#中英混输入可防止乱码\n",
    "        input.send_keys(keywords)\n",
    "        time.sleep(random.sample([1, 2, 3, 4, 5], 1)[0])\n",
    "        submit = wait.until(\n",
    "            EC.element_to_be_clickable(\n",
    "                (By.CSS_SELECTOR,\n",
    "                 '.mainv2_tab1 .inputV2 .search_button.white-btn')))\n",
    "        submit.click()\n",
    "\n",
    "        time.sleep(random.sample([1, 2, 3, 4, 5], 1)[0])\n",
    "\n",
    "        # 获取打开的多个窗口句柄\n",
    "        windows = browser.window_handles\n",
    "        time.sleep(random.sample([1, 2, 3, 4, 5], 1)[0])\n",
    "        # 切换到当前最新打开的窗口\n",
    "        browser.switch_to.window(windows[-1])\n",
    "        time.sleep(random.sample([1, 2, 3, 4, 5], 1)[0])\n",
    "\n",
    "        # 用目标元素参考去拖动\n",
    "        target_elem = browser.find_element_by_xpath(\n",
    "            '//*[@id=\"web-content\"]/div/div/div/div[1]/div[3]/div[1]/div[1]')\n",
    "        js = 'arguments[0].scrollIntoView();'\n",
    "        browser.execute_script(js, target_elem)\n",
    "\n",
    "        # 在搜索结果中获取第一个公司\n",
    "        top1 = wait.until(\n",
    "            EC.element_to_be_clickable((By.CSS_SELECTOR,\n",
    "                                        '.search_result_container a')))\n",
    "        top1.click()\n",
    "\n",
    "        # 关闭当前窗口\n",
    "        browser.close()\n",
    "\n",
    "        # 获取打开的多个窗口句柄\n",
    "        windows = browser.window_handles\n",
    "        time.sleep(random.sample([1, 2, 3, 4, 5], 1)[0])\n",
    "        # 切换到当前最新打开的窗口\n",
    "        browser.switch_to.window(windows[-1])\n",
    "        time.sleep(random.sample([1, 2, 3, 4, 5], 1)[0])\n",
    "\n",
    "        # 等待页面全部加载完成\n",
    "        wait.until(\n",
    "            EC.presence_of_all_elements_located((By.CSS_SELECTOR,\n",
    "                                                 '#_container_baseInfo')))\n",
    "        selector = etree.HTML(browser.page_source)\n",
    "\n",
    "        # 获取公司名称\n",
    "        company_name = selector.xpath(\n",
    "            '//*[@id=\"company_web_top\"]/div[2]/div[2]/div[1]/span/text()')\n",
    "        # 获取公司信息表格中的行业信息xpath //*[@id=\"_container_baseInfo\"]/div/div[3]/table/tbody/tr[3]/td[4]/text()\n",
    "        links = selector.xpath(\n",
    "            '//*[@id=\"_container_baseInfo\"]/div/div[3]/table/tbody/tr[3]/td[4]/text()'\n",
    "        )\n",
    "        time.sleep(random.sample([1, 2, 3, 4, 5], 1)[0])\n",
    "        for link in links:\n",
    "            s ='公司名称:{},找到的公司:{},所属行业:{}'.format(keywords, company_name[0],\n",
    "                                                    link)\n",
    "            print(s)\n",
    "            return s\n",
    "        # browser.close()\n",
    "    except Exception:\n",
    "        s ='公司名称:{},找到的公司:{},所属行业:{}'.format(keywords, 'NaN', 'NaN')\n",
    "        print('公司名称:{},找到的公司:{},所属行业:{}'.format(keywords, 'NaN', 'NaN'))\n",
    "        return s\n",
    "        #pass"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 36,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "#跟新数据\n",
    "def updateString(ss):\n",
    "    return ss[ss.find(',')+1:]\n",
    "\n",
    "#提取查找输入的公司名称\n",
    "def splitCompanyNameColumn(ss):\n",
    "    print(ss)\n",
    "    return ss[ss.find(':')+1:ss.find(',')]\n",
    "\n",
    "#提取找到的公司名称\n",
    "def splitFindCompanyNameColumn(ss):\n",
    "    print(ss)\n",
    "    return ss[ss.find(':')+1:ss.find(',')]\n",
    "\n",
    "#提取找到的所属行业\n",
    "def splitIndustryColumn(ss):\n",
    "    print(ss)\n",
    "    return ss[ss.find(':')+1:ss.find(',')]"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 39,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "公司名称:小米,找到的公司:小米科技有限责任公司,所属行业:科技推广和应用服务业\n",
      "公司名称:猎豹汽车研究院,找到的公司:NaN,所属行业:NaN\n",
      "公司名称:滴滴出行,找到的公司:NaN,所属行业:NaN\n",
      "0    公司名称:小米,找到的公司:小米科技有限责任公司,所属行业:科技推广和应用服务业\n",
      "1             公司名称:猎豹汽车研究院,找到的公司:NaN,所属行业:NaN\n",
      "2                公司名称:滴滴出行,找到的公司:NaN,所属行业:NaN\n",
      "Name: 临时, dtype: object\n",
      "公司名称:小米,找到的公司:小米科技有限责任公司,所属行业:科技推广和应用服务业\n",
      "公司名称:猎豹汽车研究院,找到的公司:NaN,所属行业:NaN\n",
      "公司名称:滴滴出行,找到的公司:NaN,所属行业:NaN\n",
      "0    找到的公司:小米科技有限责任公司,所属行业:科技推广和应用服务业\n",
      "1                  找到的公司:NaN,所属行业:NaN\n",
      "2                  找到的公司:NaN,所属行业:NaN\n",
      "Name: 临时, dtype: object\n",
      "找到的公司:小米科技有限责任公司,所属行业:科技推广和应用服务业\n",
      "找到的公司:NaN,所属行业:NaN\n",
      "找到的公司:NaN,所属行业:NaN\n",
      "0    所属行业:科技推广和应用服务业\n",
      "1           所属行业:NaN\n",
      "2           所属行业:NaN\n",
      "Name: 临时, dtype: object\n",
      "所属行业:科技推广和应用服务业\n",
      "所属行业:NaN\n",
      "所属行业:NaN\n"
     ]
    }
   ],
   "source": [
    "# 启动浏览器，executable_path路径要根据自己chromedriver.exe的位置更改\n",
    "browser = webdriver.Chrome(executable_path=r'/usr/local/bin/chromedriver')\n",
    "# 设置浏览器窗口位置及大小\n",
    "browser.set_window_rect(x=0, y=0, width=1200, height=750)\n",
    "# 设定页面加载限制时间\n",
    "browser.set_page_load_timeout(30)\n",
    "# 设置锁定标签等待时长\n",
    "wait = WebDriverWait(browser, 20)\n",
    "\n",
    "df = pd.read_csv('./dataset/Jam_data_analysis.csv')\n",
    "df['临时'] = df['公司'].apply(search)\n",
    "df['公司名称'] = df['临时'].apply(splitCompanyNameColumn)\n",
    "df['临时'] = df['临时'].apply(updateString)\n",
    "df['找到的公司'] = df['临时'].apply(splitFindCompanyNameColumn)\n",
    "df['临时'] = df['临时'].apply(updateString)\n",
    "df['所属行业'] = df['临时'].apply(splitIndustryColumn)\n",
    "\n",
    "#删除临时字段\n",
    "del df['临时']\n",
    "\n",
    "#关闭浏览器\n",
    "browser.close()\n",
    "\n",
    "#保存结果到csv文件中\n",
    "df.to_csv('./dataset/output.csv')"
   ]
  }
 ],
 "metadata": {
  "kernelspec": {
   "display_name": "Python 3",
   "language": "python",
   "name": "python3"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 3
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython3",
   "version": "3.6.4"
  },
  "latex_envs": {
   "LaTeX_envs_menu_present": true,
   "autoclose": false,
   "autocomplete": true,
   "bibliofile": "biblio.bib",
   "cite_by": "apalike",
   "current_citInitial": 1,
   "eqLabelWithNumbers": true,
   "eqNumInitial": 1,
   "hotkeys": {
    "equation": "Ctrl-E",
    "itemize": "Ctrl-I"
   },
   "labels_anchors": false,
   "latex_user_defs": false,
   "report_style_numbering": false,
   "user_envs_cfg": false
  },
  "varInspector": {
   "cols": {
    "lenName": 16,
    "lenType": 16,
    "lenVar": 40
   },
   "kernels_config": {
    "python": {
     "delete_cmd_postfix": "",
     "delete_cmd_prefix": "del ",
     "library": "var_list.py",
     "varRefreshCmd": "print(var_dic_list())"
    },
    "r": {
     "delete_cmd_postfix": ") ",
     "delete_cmd_prefix": "rm(",
     "library": "var_list.r",
     "varRefreshCmd": "cat(var_dic_list()) "
    }
   },
   "position": {
    "height": "328px",
    "left": "729px",
    "right": "20px",
    "top": "124px",
    "width": "350px"
   },
   "types_to_exclude": [
    "module",
    "function",
    "builtin_function_or_method",
    "instance",
    "_Feature"
   ],
   "window_display": false
  }
 },
 "nbformat": 4,
 "nbformat_minor": 2
}
