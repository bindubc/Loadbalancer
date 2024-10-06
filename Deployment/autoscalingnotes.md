To automatically spin up a new VM when the load balancer's CPU utilization exceeds 80%, you can use a combination of **monitoring tools**, **alerting**, and **Linode API automation**. Here’s how to set it up step-by-step:

### Step 1: **Set Up Monitoring for CPU Usage**
1. **Install a monitoring agent** like **Netdata** or **Prometheus** on the load balancer VMs to keep track of CPU utilization.
   
   For Netdata, you can use:
   ```bash
   bash <(curl -Ss https://my-netdata.io/kickstart.sh)
   ```

2. Use a monitoring service like **Prometheus** to collect the metrics, or use **CloudWatch (if AWS)** for easier integration.

3. Alternatively, set up **Linode Longview**, which provides detailed CPU, memory, and network metrics:
   - Install Longview:
     ```bash
     sudo apt-get update
     sudo apt-get install linode-longview -y
     ```

### Step 2: **Create an Alert for CPU Utilization**
1. Use a **monitoring tool** (e.g., Netdata, Prometheus) or set up a cron job using `top` or `mpstat` commands to periodically check CPU usage.

2. Implement a simple script on your load balancer that triggers when CPU usage crosses 80%:
   - Create a Bash script `cpu_monitor.sh`:

     ```bash
     #!/bin/bash
     CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print 100 - $1}')
     THRESHOLD=80

     if [ $(echo "$CPU_USAGE > $THRESHOLD" | bc) -eq 1 ]; then
         echo "High CPU usage detected: $CPU_USAGE%"
         # Trigger the script to create a new Linode VM
         ./create_linode_vm.sh
     else
         echo "CPU usage is normal: $CPU_USAGE%"
     fi
     ```

3. Schedule this script using **Cron** to run every minute:
   ```bash
   crontab -e
   ```
   Add the following line:
   ```
   * * * * * /path/to/cpu_monitor.sh
   ```

### Step 3: **Create a Script to Spin Up a New Linode VM**
1. Create a separate script, `create_linode_vm.sh`, that uses **Linode API** to create a new VM:

   ```bash
   #!/bin/bash

   API_TOKEN="YOUR_LINODE_API_TOKEN"
   REGION="us-east"
   IMAGE="linode/ubuntu20.04"    # Specify the OS image
   TYPE="g6-standard-2"          # Specify the type of Linode
   ROOT_PASSWORD="SecurePass123!"  # Password for the new Linode

   # Create the new Linode
   curl -X POST \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer $API_TOKEN" \
     -d '{
           "region": "'"$REGION"'",
           "type": "'"$TYPE"'",
           "image": "'"$IMAGE"'",
           "root_pass": "'"$ROOT_PASSWORD"'",
           "label": "loadbalancer-new-'$(date +%s)'",
           "tags": ["loadbalancer", "autoscaling"]
         }' \
     https://api.linode.com/v4/linode/instances

   echo "New Linode VM created successfully."
   ```

### Step 4: **Register the New VM with the Load Balancer**
1. Once a new Linode VM is created, update the **load balancer configuration** dynamically to add the new backend server.

2. Use a script to update the **Nginx** or **HAProxy** configuration:
   ```bash
   # Update Nginx upstream configuration
   sudo sed -i '/upstream backend {/a server NEW_LINODE_IP;' /etc/nginx/nginx.conf
   sudo systemctl reload nginx
   ```

3. If using **HAProxy**, add the new server dynamically:
   ```bash
   echo "server new_backend NEW_LINODE_IP:80 check" | sudo tee -a /etc/haproxy/haproxy.cfg
   sudo systemctl reload haproxy
   ```

### Step 5: **Clean Up and Remove VMs when Load Decreases**
1. Implement another script (`cleanup_vms.sh`) that checks if CPU utilization drops below a certain threshold (e.g., 30%) and **removes any extra Linode VMs**.

   ```bash
   #!/bin/bash

   # Get list of Linode VMs tagged with "autoscaling"
   LINODE_IDS=$(curl -H "Authorization: Bearer $API_TOKEN" https://api.linode.com/v4/linode/instances | jq -r '.data[] | select(.tags[] | contains("autoscaling")) | .id')

   for ID in $LINODE_IDS; do
       curl -X DELETE \
         -H "Content-Type: application/json" \
         -H "Authorization: Bearer $API_TOKEN" \
         https://api.linode.com/v4/linode/instances/$ID
       echo "Deleted Linode VM with ID: $ID"
   done
   ```

### Step 6: **Test and Validate the Solution**
1. Simulate high CPU load on your primary load balancer using a stress testing tool like `stress`:
   ```bash
   sudo apt install stress -y
   stress --cpu 4 --timeout 60
   ```
2. Ensure that the script spins up a new VM and that the load balancer configuration is updated accordingly.

3. Similarly, test the cleanup script to validate that excess VMs are removed when the load decreases.

### Optional: **Integrate with a Load Balancer Controller (e.g., Kubernetes Ingress)**
If you are using Kubernetes, consider using **Horizontal Pod Autoscalers (HPA)** to automatically scale based on CPU or memory metrics.

By following these steps, you should be able to dynamically scale your Linode VMs based on CPU utilization, ensuring high availability and performance under varying traffic loads. Let me know if you'd like more help with specific parts of the scripts!
