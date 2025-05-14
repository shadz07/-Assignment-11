# -Assignment-11
# MERN Stack Blog App Deployment with Terraform and Ansible

This document outlines the steps to deploy a MERN (MongoDB, Express.js, React, Node.js) stack blog application on AWS using Terraform for infrastructure provisioning and Ansible for backend server configuration.

## Table of Contents

1.  [Architecture Design](#1-architecture-design)
2.  [Security Configuration](#2-security-configuration)
3.  [MongoDB Atlas Setup](#3-mongodb-atlas-setup)
4.  [S3 Buckets Configuration](#4-s3-buckets-configuration)
5.  [Backend Server Setup](#5-backend-server-setup)
6.  [Frontend Deployment](#6-frontend-deployment)
7.  [Cleanup](#7-cleanup)
8.  [Success Criteria](#8-success-criteria)

## 1. Architecture Design

* A diagram illustrating the architecture is located within this repository (e.g., `architecture.png`). It details the interaction between:
    * EC2 instance (Ubuntu 22.04) for the backend
    * MongoDB Atlas cluster
    * S3 bucket for frontend hosting (static website)
    * S3 bucket for media uploads [cite: 4]

## 2. Security Configuration

* **IAM Policy and User:**
    * An IAM policy and user are created to grant programmatic access to the media upload S3 bucket. [cite: 5]
* **Security Group:**
    * A security group is configured to allow the following inbound traffic:
        * SSH (22) for server access
        * HTTP (80)
        * Application port (5000) [cite: 5]

## 3. MongoDB Atlas Setup

* The backend application uses MongoDB Atlas as the database.
* **Option A (Using existing connection string):**
    * The application may contain a default connection string in the backend's `.env` file. [cite: 5]
* **Option B (Setting up your own):**
    * If you prefer to use your own MongoDB Atlas setup:
        1.  Create a free-tier MongoDB Atlas cluster. [cite: 6, 7]
        2.  Whitelist the EC2 instance's IP address in the Atlas cluster's network settings. [cite: 6, 7]
        3.  Create a database user and obtain the connection string. [cite: 6, 7]
        4.  Update the backend's `.env` file with this connection string. [cite: 6, 7]

## 4. S3 Buckets Configuration

* **Frontend Bucket:**
    * Static website hosting is enabled for the frontend bucket. [cite: 7, 8]
    * A bucket policy is applied to allow public read access, enabling users to view the frontend. [cite: 7, 8]
* **Media Bucket:**
    * The media bucket is configured for programmatic access via IAM. [cite: 8]
    * A CORS (Cross-Origin Resource Sharing) policy is applied to allow media uploads from the application. [cite: 8]

## 5. Backend Server Setup

* Terraform is used to provision an EC2 launch template (or instance directly). [cite: 8]
    * Instance type: t3.micro
    * AMI: Ubuntu 22.04
    * The existing SSH key pair is used for secure access. [cite: 8]
* Ansible automates the backend server provisioning:
    * Ansible playbook (`backend-playbook.yml`) performs the following:
        * Clones the blog application repository from GitHub (`https://github.com/cw-barry/blog-app-MERN.git`). [cite: 8, 18, 19, 20, 21]
        * Generates and configures the `.env` file with MongoDB Atlas connection details and S3 credentials (see "Important Security Note"). [cite: 8, 18, 19, 20, 21]
        * Starts the backend application using PM2 for process management. [cite: 8, 14, 15, 16]
    * The Ansible playbook is designed to be idempotent and uses variables for sensitive data. [cite: 8, 9]
    * The playbook follows a role-based structure (see `ansible/roles/backend/`). [cite: 9]

## 6. Frontend Deployment

* Ansible is used to automate the frontend deployment. [cite: 22]
* The Ansible playbook performs the following:
    * Navigates to the frontend directory (`cd ~/blog-app/frontend`). [cite: 23]
    * Creates and configures the `.env` file with the backend API URL and media base URL. [cite: 23]
    * Configures the AWS CLI with the IAM credentials for S3. [cite: 23]
    * Installs dependencies using `pnpm`. [cite: 23]
    * Builds the frontend application (`pnpm run build`). [cite: 23]
    * Deploys the built frontend to the S3 bucket (`aws s3 sync dist/s3://<your-s3-frontend-bucket-name>/`). [cite: 23]

## 7. Cleanup

* To avoid incurring unnecessary AWS costs, use `terraform destroy` to remove all resources created by Terraform. [cite: 10]
* **Important:**
    * Remove any sensitive credentials or `.env` files from the EC2 instance. [cite: 10]
    * Delete any manually created MongoDB Atlas users or IP access rules. [cite: 10]
    * Revoke or delete IAM user credentials if created manually. [cite: 10]

## 8. Success Criteria

The deployment is considered successful if the following conditions are met:

* The MERN application is deployed and accessible in a web browser. [cite: 9]
* The backend successfully connects to the MongoDB Atlas database. [cite: 9]
* Media uploads are functional, and files are stored in the S3 media bucket. [cite: 9]
* The frontend application loads correctly from the S3 frontend bucket. [cite: 9]
* Submitted screenshots include:
    * PM2 showing the backend application running. [cite: 9]
    * MongoDB Atlas cluster status. [cite: 9]
    * Successful media upload confirmation. [cite: 9]
    * The frontend application displayed in a web browser. [cite: 9]

## Important Security Notes

* **DO NOT SHARE YOUR AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY in your submission or public repository.** [cite: 20] Handle these credentials securely using Ansible Vault or similar secure methods.
* Ensure all sensitive data (database passwords, API keys, etc.) is properly secured and not exposed in plain text.

## Helper Scripts

* The `user-data.sh` script can be used with Terraform to initialize the EC2 instance. [cite: 10, 11, 12, 13, 14, 15, 16]
* Terraform output commands can be used to retrieve S3 IAM user credentials:
    * `terraform output raw s3_user_access_key` [cite: 17]
    * `terraform output raw s3_user_secret_key` [cite: 17, 18, 19, 20, 21]

## Backend Deployment

* The `backend-deployment.sh` script provides the steps to deploy the backend application. [cite: 18, 19, 20, 21]
* These steps are converted into an Ansible playbook. [cite: 18, 19, 20, 21]

## Frontend Deployment

* The `frontend-deployment.sh` script provides the steps to deploy the frontend application. [cite: 22, 23]
* These steps are converted into an Ansible playbook. [cite: 22, 23]
