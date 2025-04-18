# AWS Announcement Scraper

Automated tool to collect AWS official announcements in Chinese and English and generate Excel files. This tool runs once a month, scrapes all AWS announcements published in the previous month, and stores them in Excel format in an S3 bucket.

## Architecture Diagram

```
┌─────────────────┐        ┌─────────────────┐        ┌─────────────────┐
│  EventBridge    │        │  AWS Lambda     │        │  S3 Bucket      │
│  Rule (Monthly) │───────>│  Function       │───────>│  Excel Files    │
└─────────────────┘        └─────────────────┘        └─────────────────┘
                                    │
                                    │                 ┌─────────────────┐
                                    └────────────────>│  SNS Topic      │
                                                      │  (Optional)     │
                                                      └─────────────────┘
```

### Component Description

- **EventBridge Rule**: Triggers execution on the 1st day of each month.
- **Lambda Function**: Runs Python 3.9 scraper code to collect AWS announcements from the previous month.
- **Lambda Layers**: Contains necessary Python dependencies (requests, BeautifulSoup4, pandas, openpyxl).
- **S3 Bucket**: Stores generated Excel files organized by year/month directory structure.
- **SNS Topic (Optional)**: Sends execution result notifications.

## Features

- Supports scraping of both Chinese and English announcements.
- Automatically identifies services available in Beijing and Ningxia regions.
- Generates Excel files with hyperlinks, click the announcement title to access the original page.
- Optional SNS notification for reporting execution success or failure.
- Fully serverless architecture, no need to maintain servers.

## File Structure

```
aws-announcement-scraper/
├── aws-announcement-scraper-cft.yaml  # CloudFormation template
├── README.md                                    # English documentation
├── examples/                                    # Example files
│   └── sample-output.xlsx                       # Sample output file
└── docs/                                        # Detailed documentation
    └── architecture.png                         # Architecture diagram
```

## Deployment Instructions

### Prerequisites

- AWS account with permissions to create CloudFormation stacks, Lambda, S3, and SNS resources.
- (Optional) SNS topic ARN for receiving notifications.

### Deployment via AWS Management Console

1. Login to AWS Management Console and navigate to CloudFormation service.

2. Click "Create Stack" > "With new resources".

3. Select "Upload a template file" and upload the `aws-announcement-scraper-cft-enhanced.yaml` file.

4. Fill in the stack name and parameters:
   - **S3BucketName**: Name of the S3 bucket where Excel files will be stored.
   - **S3KeyPrefix**: (Optional) Prefix path in the S3 bucket.
   - **LanguagePreference**: Choose the language(s) for the Excel files (en, cn, both).
   - **SNSTopicARN**: (Optional) ARN of the SNS topic for notifications.
   - **CreateS3Bucket**: Whether to create the S3 bucket.

5. Complete the remaining steps of the stack creation wizard, then click "Create Stack".

### Deployment via AWS CLI

```bash
aws cloudformation create-stack \
  --stack-name AWSAnnouncementScraper \
  --template-body file://aws-announcement-scraper-cft-enhanced.yaml \
  --parameters \
    ParameterKey=S3BucketName,ParameterValue=your-bucket-name \
    ParameterKey=S3KeyPrefix,ParameterValue=aws-announcements/ \
    ParameterKey=LanguagePreference,ParameterValue=both \
    ParameterKey=SNSTopicARN,ParameterValue=arn:aws-cn:sns:region:account-id:topic-name \
    ParameterKey=CreateS3Bucket,ParameterValue=true \
  --capabilities CAPABILITY_IAM
```

## Parameters Description

| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| S3BucketName | String | Name of the S3 bucket where Excel files will be stored. | - |
| S3KeyPrefix | String | Prefix path in the S3 bucket. | aws-announcements/ |
| LanguagePreference | String | Choose the language(s) for the Excel files. | both |
| SNSTopicARN | String | ARN of the SNS topic for notifications. | "" |
| CreateS3Bucket | String | Whether to create the S3 bucket. | false |

## Output Files

Generated Excel files will be stored at the following path:

```
s3://[S3BucketName]/[S3KeyPrefix]/[YEAR]/[MONTH]/aws_announcements_[YEAR]_[MONTH]_[LANGUAGE].xlsx
```

For example:
```
s3://my-aws-announcements/aws-announcements/2023/04/aws_announcements_2023_04_en.xlsx
s3://my-aws-announcements/aws-announcements/2023/04/aws_announcements_2023_04_cn.xlsx
```

## Excel File Format

| Launch Date | Launch Announcement | Beijing Region | Ningxia Region |
|-------------|---------------------|----------------|----------------|
| Feb 15      | [New Service Announcement](#) | GA | GA |
| Mar 20      | [Feature Update](#) |  | GA |

## Maintenance and Troubleshooting

- **View Logs**: Lambda function logs in CloudWatch Logs contain execution details.
  
- **Manual Triggering**: You can manually trigger the Lambda function via AWS console or CLI.

- **Common Issues**: If you encounter problems, check Lambda execution role permissions and S3 bucket access permissions.

## License

This project is licensed under the MIT License. See the LICENSE file for details. 
