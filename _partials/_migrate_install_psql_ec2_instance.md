## Install the psql client tools on the intermediary instance

<Procedure>

1. Connect to your intermediary EC2 instance. For example:
   ```sh
   ssh -i "<key-pair>.pem" ubuntu@<EC2 instance's Public IPv4>
   ```
1. On your intermediary EC2 instance, install the PostgreSQL client.
   ```sh
   sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
   wget -qO- https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo tee /etc/apt/trusted.gpg.d/pgdg.asc &>/dev/null
   sudo apt update
   sudo apt install postgresql-client-16 -y # "postgresql-client-16" if your source DB is using PG 16.
   psql --version && pg_dump --version
   ```

  Keep this terminal open, you need it to connect to the RDS instance for migration. 

</Procedure>

## Setup secure connectivity between your RDS and EC2 instances
<Procedure>

1. In [https://console.aws.amazon.com/rds/home#databases:](https://console.aws.amazon.com/rds/home#databases:),
    select the RDS instance to migrate.
1. Scroll down to `Security group rules (1)` and select the `EC2 Security Group - Inbound` group. The
   `Security Groups (1)` window opens. Click the `Security group ID`, then click `Edit inbound rules`

   <img class="main-content__illustration"
   src="https://assets.timescale.com/docs/images/migrate/rds-add-security-rule-to-ec2-instance.svg"
   alt="Create security group rule to enable RDS EC2 connection"/>

1. On your intermediary EC2 instance, get your local IP address:
   ```sh
   ec2metadata --local-ipv4
   ```
   Bear with me on this one, you need this IP address to enable access to your RDS instance,
1. In `Edit inbound rules`, click `Add rule`, then create a `PostgreSQL`, `TCP` rule granting access
   to the local IP address for your EC2 instance (told you :-)). Then click `Save rules`. 

   <img class="main-content__illustration"
   src="https://assets.timescale.com/docs/images/migrate/rds-add-inbound-rule-for-ec2-instance.png"
   alt="Create security rule to enable RDS EC2 connection"/>

</Procedure>

## Test the connection between your RDS and EC2 instances
<Procedure>

1. In [https://console.aws.amazon.com/rds/home#databases:](https://console.aws.amazon.com/rds/home#databases:),
    select the RDS instance to migrate.
1. On your intermediary EC2 instance, use the values of `Endpoint`, `Port`, `Master username`, and `DB name`  
   to create the postgres connectivity string to the `SOURCE` variable.
   
   <img class="main-content__illustration"
   src="https://assets.timescale.com/docs/images/migrate/migrate-source-rds-instance.svg"
   alt="Record endpoint, port, VPC details"/>

   ```sh
   export SOURCE="postgres://<Master username>:<Master password>@<Endpoint>:<Port>/<DB name>"
   ```
   The value of `Master password` was supplied when this Postgres RDS instance was created.

1. Test your connection:
   ```sh
   psql -d $SOURCE 
   ```
   You are connected to your RDS instance from your intermediary EC2 instance.

</Procedure>

[about-hypertables]: /use-timescale/:currentVersion:/hypertables/about-hypertables/
[data-compression]: /use-timescale/:currentVersion:/compression/about-compression/