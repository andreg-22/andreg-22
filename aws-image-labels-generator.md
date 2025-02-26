# AWS Image Labels Generator Project

## Project Overview
This project demonstrates how to use Amazon Rekognition to analyze images stored in Amazon S3 and generate labels for those images. The project was developed using an Ubuntu VDI on Amazon Workspaces, with AWS CLI and a Python script to interact with AWS services.

## Tools and Technologies
- **AWS Workspaces**: Used to create an Ubuntu VDI environment.
- **AWS CLI**: Installed on the Ubuntu VDI to interact with AWS services.
- **Amazon S3**: Used to store the images to be analyzed.
- **Amazon Rekognition**: Used to analyze images and generate labels.
- **Python**: Used to write the script that calls Amazon Rekognition.
- **Matplotlib**: Used to display images with bounding boxes.
- **PIL**: Used to process images.

## Steps to Reproduce

### 1. Set Up Amazon Workspaces
1. Create an Amazon Workspaces instance with an Ubuntu VDI.
2. Connect to the Ubuntu VDI using a remote desktop client.

### 2. Install AWS CLI
Open the terminal on your Ubuntu VDI and run the following commands to install AWS CLI:
```bash
sudo apt update
sudo apt install awscli -y
```

### 3. Configure AWS CLI
Use your IAM credentials to configure AWS CLI:
```bash
aws configure
```
Enter your IAM access key, secret key, default region, and output format when prompted.

### 4. Upload Images to Amazon S3
Create an S3 bucket and upload your images to the bucket. You can use the AWS Management Console or AWS CLI to do this:
```bash
aws s3 mb s3://your-bucket-name
aws s3 cp /path/to/your/images s3://your-bucket-name/ --recursive
```

### 5. Write the Python Script
Create a Python script (`label_images.py`) to analyze the images using Amazon Rekognition:

```python name=label_images.py
import boto3
import matplotlib.pyplot as plt
import matplotlib.patches as patches
from PIL import Image
from io import BytesIO

def detect_labels(photo, bucket):
    # Create a Rekognition Client
    client = boto3.client('rekognition')

    # Detect Labels in the photo
    response = client.detect_labels(
        Image={'S3Object': {'Bucket': bucket, 'Name': photo}},
        MaxLabels=10)

    # Print detected labels
    print('Detected labels for ' + photo)
    print()
    for label in response['Labels']:
        print("Label:", label['Name'])
        print("Confidence:", label['Confidence'])
        print()

    # Load the image from S3
    s3 = boto3.resource('s3')
    obj = s3.Object(bucket, photo)
    img_data = obj.get()['Body'].read()
    img = Image.open(BytesIO(img_data))

    # Display the image with bounding boxes
    plt.imshow(img)
    ax = plt.gca()
    for label in response['Labels']:
        for instance in label.get('Instances', []):
            bbox = instance['BoundingBox']
            left = bbox['Left'] * img.width
            top = bbox['Top'] * img.height
            width = bbox['Width'] * img.width
            height = bbox['Height'] * img.height
            rect = patches.Rectangle((left, top), width, height, linewidth=1, edgecolor='r', facecolor='none')
            ax.add_patch(rect)
            label_text = label['Name'] + ' (' + str(round(label['Confidence'], 2)) + '%)'
            plt.text(left, top - 2, label_text, color='r', fontsize=8, bbox=dict(facecolor='white', alpha=0.7))
    plt.show()

    return len(response['Labels'])

def main():
    photo = 'kitchen-1940177_1920.jpg'
    bucket = 'image-id-for-ai-testing'
    label_count = detect_labels(photo, bucket)
    print("Labels detected:", label_count)

if __name__ == "__main__":
    main()
```

### 6. Install Required Python Libraries
Install the required Python libraries using `pip`:
```bash
pip install boto3 matplotlib pillow
```

### 7. Run the Python Script
Execute the Python script from the terminal:
```bash
python3 label_images.py
```

## Project Results
The script will output the detected labels and their confidence scores for each image. Here is an example of the output:
```
Detected labels for kitchen-1940177_1920.jpg

Label: Kitchen
Confidence: 99.9

Label: Appliance
Confidence: 96.7

...
```

Additionally, the script will display the image with bounding boxes around detected objects and their labels.

## Conclusion
This project demonstrates how to use Amazon Rekognition to analyze images stored in Amazon S3 and generate labels for those images. By following the steps outlined above, you can reproduce this project and extend it further based on your requirements.