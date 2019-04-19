# Install Kappbot

Download:
```bash
git clone https://github.com/Ptomerty/Kappbot.git
cd Kappbot
git checkout dev
npm install
npm install -g forever
sudo ln -s $(which forever) /usr/local/bin/forever
```

Drop in `~/start.sh` file:
```bash
#!/bin/sh

cd ~/Kappbot
forever start Kappbot.sh
```