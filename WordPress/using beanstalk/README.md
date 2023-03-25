
# Deploying a high-availability WordPress website with an external Amazon RDS database to Elastic Beanstalk

## Launch a DB instance in Amazon RDS

When you launch an instance with Amazon RDS, it's completely independent of Elastic Beanstalk and your Elastic Beanstalk environments, and will not be terminated or monitored by Elastic Beanstalk.

In the following steps you'll use the Amazon RDS console to:

1. Launch a database with the MySQL engine.

2. Enable a Multi-AZ deployment. This creates a standby in a different Availability Zone (AZ) to provide data redundancy, eliminate I/O freezes, and minimize latency spikes during system backups.

## To launch an RDS DB instance in a default VPC

1.  Open the RDS console.

2. In the navigation pane, choose Databases.

3. Choose Create database.

4. Choose Standard Create.

5. Under Additional configuration, for Initial database name, type ebdb.

6. Review the default settings and adjust these settings according to your specific requirements. Pay attention to the following options:

* DB instance class – Choose an instance size that has an appropriate amount of memory and CPU power for your workload.

* Multi-AZ deployment – For high availability, set this to Create an Aurora Replica/Reader node in a different AZ.

* Master username and Master password – The database username and password. Make a note of these settings because you will use them later.

7. Verify the default settings for the remaining options, and then choose Create database.


* After your DB instance is created, modify the security group attached to it in order to allow inbound traffic on the appropriate port..

## To modify the inbound rules on the security group that's attached to your RDS instance

1. Open the Amazon RDS console.

2. Choose Databases.

3. Choose the name of your DB instance to view its details.

4. In the Connectivity section, make a note of the Subnets, Security groups, and Endpoint that are displayed on this page. This is so you can use this information later.

5. Under Security, you can see the security group that's associated with the DB instance. Open the link to view the security group in the Amazon EC2 console.

6.  In the security group details, choose Inbound.

7.   Choose Edit.

8. Choose Add Rule.

9.  For Type, choose the DB engine that your application uses.

10. For Source, type sg- to view a list of available security groups. Choose the security group that's associated with the Auto Scaling group that's used with your Elastic Beanstalk environment. This is so that Amazon EC2 instances in the environment can have access to the database.

11. Choose Save.

Creating a DB instance takes about 10 minutes. In the meantime, download WordPress and create your Elastic Beanstalk environment.

## Download WordPress

https://wordpress.org/latest.zip

1.  Extract the zip file .

2.  Open wordpress folder select all the files and compress it and make a zip file.

3. Use this zip file to upload on Elastic Beanstalk.


## Launch an Elastic Beanstalk environment

Use the Elastic Beanstalk console to create an Elastic Beanstalk environment. After you launch the environment, you can configure it to connect to the database, then deploy the WordPress code to the environment.

In the following steps, you'll use the Elastic Beanstalk console to:

1.  Create an Elastic Beanstalk application using the managed PHP platform.

2. Accept the default settings and sample code.

#### To launch an environment (console)

1. Open the Elastic Beanstalk console

2. For Platform, select the platform and platform branch that match the language used by your application.

3. For Application code, choose Sample application.

4. Choose Review and launch.

5. Review the available options. Choose the available option you want to use, and when you're ready, choose Create app.

* All of these resources are managed by Elastic Beanstalk. When you terminate your environment, Elastic Beanstalk terminates all the resources that it contains.

* Because the Amazon RDS instance that you launched is outside of your environment, you are responsible for managing its lifecycle.

## Configure security groups and environment properties

* Add the security group of your DB instance to your running environment. This procedure causes Elastic Beanstalk to reprovision all instances in your environment with the additional security group attached.

#### To add a security group to your environment

1. Open the Elastic Beanstalk console, and in the Regions list, select your AWS Region.

2. In the navigation pane, choose Environments, and then choose the name of your environment from the list.

3. In the navigation pane, choose Configuration.

4.  In the Instances configuration category, choose Edit.

5. Under EC2 security groups, choose the security group to attach to the instances, in addition to the instance security group that Elastic Beanstalk creates.

6. Choose Apply at the bottom of the page.

7. Read the warning, and then choose Confirm.

#### To configure environment properties for an Amazon RDS DB instance

1. Open the Elastic Beanstalk console, and in the Regions list, select your AWS Region.

2. In the navigation pane, choose Environments, and then choose the name of your environment from the list.

3. In the navigation pane, choose Configuration.

4. In the Software configuration category, choose Edit.

5. In the Environment properties section, define the variables that your application reads to construct a connection string. For compatibility with environments that have an integrated RDS DB instance, use the following names and values. You can find all values, except for your password, in the RDS console
 











| Property name | Description    | Property value               |
| :-------- | :------- | :------------------------- |
| RDS_HOSTNAME| The hostname of the DB instance. | On the Connectivity & security tab on the Amazon RDS console: Endpoint.
 RDS_PORT |The port where the DB instance accepts connections. |On the Connectivity & security tab on the Amazon RDS console: Port.
 |RDS_DB_NAME| The database name, ebdb. | On the Configuration tab on the Amazon RDS console: DB Name.
 | RDS_USERNAME| The username that you configured for your database. |On the Configuration tab on the Amazon RDS console: Master username.
 | RDS_PASSWORD| The password that you configured for your database. |  Put your  RDS password


6. Choose Apply at the bottom of the page.

## To deploy WordPress

1.  Open the Elastic Beanstalk console

2.  On the environment overview page, choose Upload and deploy.

3.  Use the on-screen dialog box to upload the source bundle.

4.  Choose Deploy

5.  When the deployment completes, you can choose the site URL to open your website in a new tab.


## Clean up

####  Terminate your Elastic Beanstalk environment

#### Terminate your RDS DB



