## Master In 3-Tier Architecture || Ultimate Guide For DevOps Engineer

Problem Statement:
-------------------
Design a scalable, secure, and high-availability 3-tier architecture for an e-commerce web application on AWS. 
The architecture should support dynamic content, handle peak loads efficiently, 
and ensure data integrity and security. Maintain Backup & Disaster Recovery.

 - Scalability & Availability:
	The application should be able to handle sudden spikes in traffic, especially during sales and promotions.
	Ensure high availability and fault tolerance to provide an uninterrupted user experience.

 - Security:
	Ensure secure communication between different layers and components of the architecture.
	Protect sensitive user data, including personal and payment information.
	Implement security best practices for data at rest and in transit.

 - Cost-effectiveness:
	Optimize the architecture for cost without compromising performance and security.
	
 - Performance:
	Optimize for low latency and high throughput.
	Ensure the application can deliver a responsive and fast user experience.
	For global user best performance experience

- Layers:

1. Web - Serve static and dynamic content to users.
2. App - Handle business logic and process user requests.
3. DB  - Store and manage application data securely and efficiently.    

Tech Stack:
-----------

- Web Layer	( FrontEnd )
	- AWS EC2 ( In Private Subnet )
	- Load Balancer - Public
	- CloudFront
	- AWS S3 Bucket
	- AWS Route53
	- AWS ACM - Certificate Manager
	- Auto Scaling
	
- Application Layer ( Middle handle logical operations )
	- AWS EC2 ( In Private )
	- Load Balancer - Internal
	- Auto Scaling
	
- Database Layer ( Backend )
		- AWS RDS - MySQL