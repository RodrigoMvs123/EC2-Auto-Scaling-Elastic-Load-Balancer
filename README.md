# EC2-Auto-Scaling-Elastic-Load-Balancer

https://www.youtube.com/watch?v=cf9jQc4xzpo

##

https://aws.amazon.com/pt/ec2/autoscaling/

##
dynamic-website-scaling

const fs = require('node:fs/promises');

const express = require('express');
const bodyParser = require('body-parser');

const app = express();

app.use(bodyParser.json());

app.get('/', async function (req, res) {
  let dummyData;
  for (let i = 0; i < 5000; i++) {
    dummyData = await fs.readFile('data.json');
  }
  const json = JSON.parse(dummyData);
  res.json({ message: 'Success!', data: json });
});

app.post('/data', async function (req, res) {
  const data = req.body;

  res.status(201).json({ message: 'Received dummy data.', data });
});

app.listen(3000);

##

Send-request.py
from multiprocessing import pool
from urllib import response

from requests import get 

DOMAIN = 'ec2-3-86-207-211.compute-1.amazonaws.com'

def send_request(val):
    while True:
        response = get(f'https://{DOMAIN}')
        data = response.json()
        print('Sent request')
        print(data)

if __name__ == '__main__':
    with Pool(150) as p:
        p.map(send_request, range(150))
##

userdata-with-caching.sh
#!/bin/bash

# Enable logs
exec > >(tee /var/log/user-data.log|logger -t user-data -s 2>/dev/console) 2>&1

# Install Git
echo "Installing Git"
yum update -y
yum install git -y

# Install NodeJS
echo "Installing NodeJS"
touch .bashrc
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash
. /.nvm/nvm.sh
nvm install --lts

# Clone website code
echo "Cloning website"
mkdir -p /demo-website
cd /demo-website
git clone https://github.com/academind/aws-demos.git .
cd dynamic-website-with-cloudfront

# Install dependencies
echo "Installing dependencies"
npm install

# Forward port 80 traffic to port 3000
echo "Forwarding 80 -> 3000"
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-ports 3000

# Install & use pm2 to run Node app in background
echo "Installing & starting pm2"
npm install pm2@latest -g
pm2 start app.js
