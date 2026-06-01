# typora_image_upload_to_cos
Python Script for Uploading Images from Typora to Tencent Cloud COS
# Typora设置与对象存储说明

## 对象存储

搭配Typora的图床是腾讯云COS（对象存储），腾讯云COS购买**COS 低频存储容量包**和**COS 标准存储请求包**

安装Typroa以后，在偏好设置中进行全局图像上传设置，插入图片时 本地 和 网络 图片按都上传规则。

## 腾讯COS操作

购买以后设置COS，在 CAM 控制台访问管理中可创建子账号，并配置授予子账号的访问权限。具体操作如下所示：

登录**控制台**进入 **账户** 下面的 **访问管理**。
选择【用户】>【用户列表】>【新建用户】，进入新建用户页面。
选择【自定义创建】，选择【可访问资源并接收消息】类型，单击【下一步】。
按照要求填写用户相关信息。
设置用户信息：输入子用户名称，例如 Sub_user。输入子用户的邮箱，您需要为子用户添加邮箱来获取由腾讯云发出的绑定微信的邮件。
访问方式：选择编程访问和腾讯云控制台访问。其他配置可按需选择。
填写用户信息完毕后，单击【下一步】，进行身份验证。
身份验证完毕，设置子用户权限。根据系统提供的策略选择，可配置简单的策略，例如 COS 的存储桶列表的完全访问权限 等。单击【完成】即可创建子账号。API密钥在用户列表子账户的账户详情中。

## 上传图片到腾讯云的COS存储桶中python程序

为了将图片上传到腾讯云对象存储（COS），我们可以使用腾讯云提供的 Python SDK `cos-python-sdk-v5`。以下是一个示例程序，它展示了如何使用该 SDK 将图片上传到腾讯云 COS，并生成 Markdown 格式的图片链接。

首先，你需要在腾讯云控制台上创建一个 COS Bucket，并获取相关的密钥信息（SecretId 和 SecretKey）。然后，安装腾讯云 COS SDK：

在windows系统的命令行CMD或者PowerShell中运行安装。

```bash
pip install cos-python-sdk-v5
```

在 **Windows 系统**中设置环境变量有两种主要方法：通过命令行（CMD/PowerShell）或通过系统设置界面。以下是这两种方法的详细步骤：

### 方法一：通过命令行设置环境变量

1. **打开命令提示符（CMD）**：

   - 按 `Win + R`，输入 `cmd`，然后按 `Enter`。

2. **使用 `setx` 命令设置环境变量**：

   - 输入以下命令并按 Enter

     ```cmd
     shCopy codesetx COS_SECRET_ID "your_secret_id"
     setx COS_SECRET_KEY "your_secret_key"
     setx COS_REGION "your_region"
     setx COS_BUCKET "your_bucket"
     ```

或者在系统属性中直接设置，注意设置后重启生效。

### 方法二：通过系统设置界面设置环境变量（推荐）

1. **打开系统属性**：

   - 按 `Win + X` 并选择“系统”。
   - 点击“高级系统设置”。
   - 在“系统属性”窗口中，点击“环境变量”。

2. **设置系统环境变量**：

   - 在“环境变量”窗口中，选择“用户变量”或“系统变量”部分，然后点击“新建”。
   - 在“新建用户变量”或“新建系统变量”窗口中，输入变量名和变量值，然后点击“确定”。重复此步骤为每个变量设置。

   例如：

   - 变量名：`COS_SECRET_ID` 变量值：`your_secret_id`
   - 变量名：`COS_SECRET_KEY` 变量值：`your_secret_key`
   - 变量名：`COS_REGION` 变量值：`your_region`
   - 变量名：`COS_BUCKET` 变量值：`your_bucket`

3. **保存设置并关闭所有窗口**：

   - 点击“确定”保存每个环境变量。
   - 最后点击“确定”关闭“环境变量”窗口和“系统属性”窗口。
   - 注意设置后重启生效。

### 检查环境变量

可以在命令提示符中运行以下命令来检查环境变量是否已正确设置：

```cmd
echo %COS_SECRET_ID%
echo %COS_SECRET_KEY%
echo %COS_REGION%
echo %COS_BUCKET%
```

### 上传图片python程序

将upload_to_cos.py文件放在 "D:/Program Files (x86)/" 文件夹下

```python
import os
import sys
from qcloud_cos import CosConfig
from qcloud_cos import CosS3Client
import argparse
import logging

logging.basicConfig(level=logging.WARNING, stream=sys.stdout)  # 输出日志警告信息

def get_content_type(file_name):
    ext = os.path.splitext(file_name)[1].lower()
    if ext == '.jpg' or ext == '.jpeg':
        return 'image/jpeg'
    elif ext == '.png':
        return 'image/png'
    elif ext == '.gif':
        return 'image/gif'
    elif ext == '.bmp':
        return 'image/bmp'
    else:
        return 'application/octet-stream'  # 默认值

def upload_to_cos(secret_id, secret_key, region, bucket_name, local_file_path):
    config = CosConfig(Region=region, SecretId=secret_id, SecretKey=secret_key)
    client = CosS3Client(config)
    
    file_name = os.path.basename(local_file_path)
    upload_path = f"noteimages/{file_name}"  # 上传路径包含文件夹
    # 因腾讯存储桶必须上传图片到文件夹中才能压缩打包下载，所以设置图片上传路径到文件夹。
    content_type = get_content_type(file_name)

    with open(local_file_path, 'rb') as f:
        response = client.put_object(
            Bucket=bucket_name,
            Body=f,
            Key=upload_path,
            StorageClass='STANDARD',
            ContentType=content_type
        )

    file_url = f"https://{bucket_name}.cos.{region}.myqcloud.com/{upload_path}"
    return file_url

def main():
    parser = argparse.ArgumentParser(description="Upload images to Tencent Cloud COS and generate Markdown links")
    parser.add_argument('--file', required=True, nargs='+', help="Local image file paths")

    args = parser.parse_args()

    secret_id = os.getenv('COS_SECRET_ID')
    secret_key = os.getenv('COS_SECRET_KEY')
    region = os.getenv('COS_REGION')
    bucket = os.getenv('COS_BUCKET')

    if not all([secret_id, secret_key, region, bucket]):
        print("Please set the environment variables: COS_SECRET_ID, COS_SECRET_KEY, COS_REGION, COS_BUCKET")
        sys.exit(1)

    for file_path in args.file:
        file_url = upload_to_cos(secret_id, secret_key, region, bucket, file_path)
        markdown_link = f"![{os.path.basename(file_path)}]({file_url})"
        print(markdown_link)

if __name__ == "__main__":
    main()
```

### 测试

在 Typora 中，打开 `File -> Preferences -> Image`（如果你使用的是中文版，路径是 `文件 -> 偏好设置 -> 图像`）。

上传服务：自定义命令

命令框中填写下面代码：

```python
python "D:/Program Files (x86)/upload_to_cos.py" --file
```

验证图片上传只返回Markdown的图片格式代码才正确，否则错误。

如 `![image](https://typro-123456789.cos.ap-beijing.myqcloud.com/images/image-20260531015844916.png)`
