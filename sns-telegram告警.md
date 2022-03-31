# 1. 创建sns主题  
- 转到AWS管理控制台>简单通知服务>主题> 创建主题。给它起一个名字，然后单击“创建主题”，我这里选择标准类型。  
# 2. 创建lambda函数  
2.1 创建函数  
- 转到AWS管理控制台> Lambda> 函数 >创建函数。给函数命名，然后选择Python 3.7运行时。  
注：python3.8已经从botocore.vendored.requests模块中移除了post方法！   
2.2 配置环境变量   
- 配置 > 环境变量 > 填写机器人token和群组id    
![avatar](https://raw.githubusercontent.com/tanxw123123/aws/master/picture/01.jpg)  
2.3 编写代码（通过代码发送到telegram告警）  
- 编辑好代码点击deploy保存！  
```
import json
import os
import logging
from botocore.vendored import requests

# Initializing a logger and settign it to INFO
logger = logging.getLogger()
logger.setLevel(logging.INFO)

# Reading environment variables and generating a Telegram Bot API URL
TOKEN = os.environ['TOKEN']
USER_ID = os.environ['USER_ID']
TELEGRAM_URL = "https://api.telegram.org/bot{}/sendMessage".format(TOKEN)

# Helper function to prettify the message if it's in JSON
def process_message(input):
    try:
        # Loading JSON into a string
        raw_json = json.loads(input)
        # Outputing as JSON with indents
        output = json.dumps(raw_json, indent=4)
    except:
        output = input
    return output

# Main Lambda handler
def lambda_handler(event, context):
    # logging the event for debugging
    logger.info("event=")
    logger.info(json.dumps(event))

    # Basic exception handling. If anything goes wrong, logging the exception    
    try:
        # Reading the message "Message" field from the SNS message
        message = process_message(event['Records'][0]['Sns']['Message'])

        # Payload to be set via POST method to Telegram Bot API
        payload = {
            "text": message.encode("utf8"),
            "chat_id": USER_ID
        }

        # Posting the payload to Telegram Bot API
        requests.post(TELEGRAM_URL, payload)

    except Exception as e:
        raise e

```
2.4 添加触发器  
选择SNS，选择我们上面创建的sns主题  

# 3. 测试通过sns主题发布消息  

