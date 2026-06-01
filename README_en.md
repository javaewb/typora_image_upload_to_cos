# typora_image_upload_to_cos

Python Script for Uploading Images from Typora to Tencent Cloud COS

# Typora Settings and Object Storage Instructions

## Object Storage

This project uses Tencent Cloud COS (Cloud Object Storage) as the image hosting service for Typora. You should purchase the COS storage package (for example, COS Infrequent Access package) and the COS standard storage request package as needed.

After installing Typora, configure the global image upload settings in Preferences. When inserting images, both local and remote images follow the upload rule you've set.

## Tencent COS setup

After purchasing COS, configure it in the Tencent Cloud console. In the CAM (Cloud Access Management) console, you can create sub-accounts and grant them the required permissions. General steps:

1. Log in to the console and go to Account -> Access Management.
2. Select "Users" > "User List" > "Create User".
3. Choose "Custom create" and select the access type "Accessible resources and receive messages", then click Next.
4. Fill in the user's information as required.
5. Set the user details: enter a sub-user name (for example, Sub_user) and the sub-user's email (needed to receive binding emails from Tencent Cloud).
6. Choose access methods: enable Programmatic Access and Tencent Cloud Console Access as needed.
7. After filling the information, click Next for identity verification.
8. After verification, assign permissions to the sub-user. Choose appropriate policies, for example, full access to the COS bucket list for testing.
9. Click Finish to create the user.

(Assign permissions according to your security requirements; the above is a simple example.)

## Python program to upload images to a Tencent COS bucket

To upload images to Tencent Cloud Object Storage (COS), you can use Tencent Cloud's Python SDK `cos-python-sdk-v5`. The example below shows how to use the SDK to upload an image.

First, create a COS Bucket in the Tencent Cloud console and obtain the necessary keys (SecretId and SecretKey). Then install the COS SDK:

Run the install command in CMD or PowerShell on Windows:

```bash
pip install cos-python-sdk-v5
```

### Set environment variables on Windows

There are two main ways to set environment variables on Windows: via the command line (CMD/PowerShell) or via the System Settings UI. Both methods are shown below.

#### Method 1: Set environment variables via the command line

1. Open Command Prompt (CMD):
   - Press Win + R, type `cmd`, press Enter.

2. Use `setx` to create environment variables:

```cmd
setx COS_SECRET_ID "your_secret_id"
setx COS_SECRET_KEY "your_secret_key"
setx COS_REGION "your_region"
setx COS_BUCKET "your_bucket"
```

Note: You may need to restart your terminal or log out/in for the variables to take effect system-wide.

#### Method 2: Set environment variables via System Settings (recommended)

1. Open System Properties:
   - Press Win + X and select "System".
   - Click "Advanced system settings".
   - In the System Properties window, click "Environment Variables".

2. Add system/user environment variables:
   - In the Environment Variables window, choose either "User variables" or "System variables" and click "New".
   - Enter the variable name and value, then click OK. Repeat for each variable.

   Example:
   - Variable name: `COS_SECRET_ID` Value: `your_secret_id`
   - Variable name: `COS_SECRET_KEY` Value: `your_secret_key`
   - Variable name: `COS_REGION` Value: `your_region`
   - Variable name: `COS_BUCKET` Value: `your_bucket`

3. Save and close:
   - Click OK to save each dialog. Restart may be required for changes to apply.

### Check environment variables

You can check environment variables in the command prompt:

```cmd
echo %COS_SECRET_ID%
echo %COS_SECRET_KEY%
echo %COS_REGION%
echo %COS_BUCKET%
```

### Upload script usage

Place the `upload_to_cos.py` (or `uploadtocos.py`) file where you want to run it, for example "D:/Program Files (x86)/".

Example Python script usage (the script prints the uploaded file URL or markdown link depending on configuration):

```python
# Example snippet (see full script file in repository)
python "D:/Program Files (x86)/upload_to_cos.py" --file <image_path>
```

### Test in Typora

In Typora, open File -> Preferences -> Image (or in Chinese: 文件 -> 偏好设置 -> 图像).

Set Upload Service to: Custom Command

In the command box, enter something like:

```text
python "D:/Program Files (x86)/upload_to_cos.py" --file
```

Typora expects the upload command to return either a URL or a Markdown image reference. If the upload script returns the Markdown image format, Typora will insert it correctly. Example of a correct return value:

![image](https://your-bucket.cos.region.myqcloud.com/path/to/image.png)