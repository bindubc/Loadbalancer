# Loadbalancer Hackathon Assignment

### Project Name: High-Availability Load Balancer

### Description
Welcome to the repository for the High-Availability Load Balancer project, created during the Akamai Hackathon, Oct 2024. Our project aims to set up a robust and scalable load balancer that meets the specifications for traffic management, performance, reliability, logging, security, and programmatic interaction, ensuring high availability for public-facing applications.

### Key Requirement


* Allow traffic only on Port 443 (HTTPS).
* Valid SSL Certificate to be deployed on Port 443 with automated certificate renewal.
* Gracefully handle connections on Port 80 (HTTP) and redirect traffic to Port 443 (HTTPS).
* Ability to handle 50,000 concurrent connections.
* Automated failover between primary and secondary load balancer VMs (or vice versa).
* Logging of all activities on the load balancer for a minimum of 1 day, with purging of logs older than 1 day.
* Securing load balancer VM (VM Hardening).
* Showcase programmatic interaction with the load balancer VM using Linode APIs.
* The load balancer should have the capability to define rules at Layer 4 (Network layer) and Layer 7 (Application layer):
    * At Layer 4: Ability to allow/deny requests based on IP address.
    * At Layer 7: Ability to look at HTTP cookies and ensure session stickiness is maintained.

### Features
- **Traffic Management**: 
  - Allow traffic only on Port 443 (HTTPS) with a valid SSL certificate deployed and automated renewal via Let's Encrypt.
  - Gracefully handle connections on Port 80 (HTTP) and redirect all traffic to Port 443.

- **Performance and Reliability**: 
  - Support up to 50,000 concurrent connections by creating a resilient application server.
  - Implement automated failover between primary and secondary load balancer VMs using Keepalived and HAProxy.

- **Logging and Security**: 
  - Log all activities on the load balancer for at least one day, with automated log purging for logs older than one day.
  - Secure the load balancer VM through proper hardening practices, including firewall configurations and SSH authentication.

- **Programmatic Interaction**: 
  - Showcase interaction with the load balancer VM using Linode APIs.

- **Layered Rules**: 
  - Implement Layer 4 rules to allow/deny requests based on IP addresses.
  - Implement Layer 7 rules to examine HTTP cookies and maintain session stickiness.

### Technologies Used
- Linode (for VM deployment)
- HAProxy (for load balancing)
- Keepalived (for failover management)
- Let's Encrypt (for SSL certificates)
- [Additional technologies as needed,[### TBD] ]

### Installation <tentaively> 
To get started with this project, follow these steps:
1. Clone the repository: `git clone [repository URL]`
2. Install dependencies: `[installation command, e.g., npm install or pip install -r requirements.txt]`
3. Configure the load balancer settings in `config.yaml` (or appropriate config file).
4. Deploy the application: `[run command]`



### Team Members
- Bindu BC - Software Engineer
- Khagesh Kumar - Software Engineer
- Mohit Sahu - Software Engineer

### Acknowledgments
Special thanks to Akamai Hackathon Team


