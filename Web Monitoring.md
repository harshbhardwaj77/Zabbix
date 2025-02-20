# Monitoring `https://www.google.com` with Zabbix Web Scenarios

## Overview

Zabbix's **Web Monitoring** feature allows you to simulate user interactions with websites, ensuring they are accessible and performing as expected. By configuring a **Web Scenario**, you can monitor various metrics such as response time, download speed, and the presence of specific content on a webpage. This guide demonstrates how to set up a Web Scenario in Zabbix to monitor `https://www.google.com`.

## Prerequisites

Before configuring web monitoring in Zabbix, ensure the following:

- **Zabbix Server with cURL Support**: Zabbix must be compiled with cURL (libcurl) support to perform web monitoring.
- **Access Permissions**: You should have administrative privileges to configure hosts and templates in Zabbix.

## Step 1: Create a Host for the Website

1. **Navigate to Hosts Configuration**:
   - Go to `Configuration` → `Hosts` in the Zabbix frontend.

2. **Create a New Host**:
   - Click on `Create host` in the upper-right corner.
   - Fill in the following details:
     - **Host Name**: `Google Website`
     - **Visible Name**: (Optional) `Google`
     - **Groups**: Select an existing group or create a new one, e.g., `Websites`.
     - **Interfaces**: No need to define an interface for web monitoring.
   - Click `Add` to create the host.

## Step 2: Create a Web Scenario

1. **Access the Web Scenarios Configuration**:
   - In the `Hosts` configuration, find the `Google Website` host you just created.
   - Click on `Web` in the row corresponding to the host.
   - Click on `Create web scenario` to add a new scenario.

2. **Define the Web Scenario Parameters**:
   - **Name**: `Google Homepage Monitoring`
   - **Update interval**: Specify how often the scenario should run, e.g., `1m` for every minute.
   - **Attempts**: Number of retry attempts in case of a failure, e.g., `3`.
   - **Agent**: Choose a user agent string to simulate, e.g., `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.3`.
   - **Steps**: Define the steps to be performed in this scenario.

3. **Configure the Step for Monitoring `https://www.google.com`**:
   - Click on `Add` to create a new step.
   - Fill in the following details:
     - **Name**: `Access Google Homepage`
     - **URL**: `https://www.google.com`
     - **Timeout**: Time to wait for a response, e.g., `15s`.
     - **Required status codes**: Specify expected HTTP response codes, e.g., `200`.
     - **Required string**: (Optional) Specify a string that should be present in the response, e.g., `Google Search`.
     - **Post**: (Leave empty for a GET request)
     - **Headers**: (Optional) Add any custom headers if required.
   - Click `Add` to save the step.

4. **Save the Web Scenario**:
   - After adding all necessary steps, click `Add` to save the web scenario.

## Step 3: Creating Triggers for the Web Scenario

To receive alerts based on the web scenario's performance, set up triggers.

1. **Navigate to Triggers Configuration**:
   - Go to `Configuration` → `Hosts`.
   - Click on the `Triggers` link for the `Google Website` host.

2. **Create a New Trigger**:
   - Click on `Create trigger`.
   - Fill in the following details:
     - **Name**: `Google Homepage is Unavailable`
     - **Severity**: Choose an appropriate level, e.g., `High`.
     - **Expression**: Use the following expression to trigger an alert if the web scenario fails:
       ```
       {Google Website:web.test.fail[Google Homepage Monitoring].last()}<>0
       ```
       This expression evaluates whether the last run of the web scenario resulted in a failure.
   - Click `Add` to save the trigger.

## Step 4: Setting Up Notifications

To be notified when the trigger activates, configure actions in Zabbix.

1. **Navigate to Actions Configuration**:
   - Go to `Configuration` → `Actions`.

2. **Create a New Action**:
   - Click on `Create action`.
   - Fill in the following details:
     - **Name**: `Notify Admins on Google Homepage Issues`
     - **Conditions**: Set conditions to specify when this action should be triggered, e.g., when the trigger `Google Homepage is Unavailable` is activated.
     - **Operations**: 
::contentReference[oaicite:0]{index=0}
 
