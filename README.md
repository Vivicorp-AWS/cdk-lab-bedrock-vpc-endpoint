# CDK Lab - Use Bedrock in Network Isolated Environment with VPC Endpoint

## Overview

This project demonstrates how to access Amazon Bedrock from an EC2 instance in a completely isolated network environment (no internet access) using VPC endpoints. The EC2 instance is deployed in a private isolated subnet and connects to Bedrock Runtime through an Interface VPC Endpoint, ensuring all traffic stays within the AWS network.

This project extends the existing [cdk-lab-eice](https://github.com/Vivicorp-AWS/cdk-lab-eice) project, which demonstrates EC2 Instance Connect Endpoint (EICE) usage. This version adds Amazon Bedrock integration to show how to use AWS AI services in network-isolated environments.

## Resources Created

This CDK stack creates the following AWS resources:

- **VPC**: A Virtual Private Cloud with 2 Availability Zones
  - 2 Private Isolated Subnets (no internet gateway or NAT gateway)
- **VPC Endpoint**: Interface VPC Endpoint for Bedrock Runtime service with private DNS enabled
- **Security Groups**:
  - EICE Security Group (for Instance Connect Endpoint)
  - EC2 Security Group (for the bastion instance)
- **IAM Role**: BastionRole with AmazonBedrockFullAccess policy attached
- **EC2 Instance**: t3.micro Amazon Linux 2023 instance in private isolated subnet
- **EC2 Instance Connect Endpoint (EICE)**: Enables SSH access to the private EC2 instance without requiring a public IP or bastion host with internet access

## Usage

### Deploy

```bash
cdk deploy \
  --all \
  --require-approval=never \
  --outputs-file ./cdk.out/outputs.json
```

### Connect to the Bastion EC2 Instance

```bash
aws ec2-instance-connect ssh \
  --instance-id $(jq -r ".EICEStack.EC2InstanceID" ./cdk.out/outputs.json) \
  --os-user ec2-user
```

### Make inference to Bedrock Anthropic Claude Haiku 4.5 Model

```bash
# Generate payload file
cat > payload.json <<EOF
{
    "modelId": "us.anthropic.claude-3-5-haiku-20241022-v1:0",
    "messages": [{"role": "user", "content": [{"text": "${1:-Hello}"}]}],
    "inferenceConfig": {"maxTokens": 2048, "temperature": 1.0}
}
EOF

# Make inference call
aws bedrock-runtime converse --cli-input-json file://payload.json
```

### Clean up

```bash
cdk destroy --all
```