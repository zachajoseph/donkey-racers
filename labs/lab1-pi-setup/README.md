# Lab 1: Time to Get Pied!
 
- [Introduction](#introduction)
- [Part I: Sharing the Pie](#part-i-sharing-the-pie)
- [Part II: Team Selfie!](#part-ii-team-selfie)
- [Part III: Time to be a detect-ive!](#part-iii-time-to-be-a-detect-ive)
- [Conclusion](#conclusion)
- [Where This Goes](#where-this-goes)
---
 
## Introduction
 
Welcome to your first lab! Lecture resources are at the [Donkey Racers Plans](https://docs.google.com/spreadsheets/d/1WbkyzSDLT9U-AC8TgL6pEixj6dHXa7hKu6J-1AbAtmc/edit?gid=1152629144#gid=1152629144) link.
 
In lecture, we introduced **GitHub**, the **terminal**, and the Raspberry Pi. By the end of this lab, every teammate will have their own account on your team's Pi, connected to GitHub, with the Python environment your car will run on. You'll also test the camera and your first I2C device.
 
Try to work through problems yourself first, but don't struggle unnecessarily — the Internet, AI tools, and both leads are here to help.
 
> **AI usage:** AI tools are great for debugging and understanding errors. Just make sure you can explain every command you run — see `CONTRIBUTING.md` for the full policy.
 
---
 
## Part I: Sharing the Pie
 
Your team's Pi comes preflashed with the OS, camera tools, and I2C enabled. In this part, you'll give every teammate their own account, connect each account to GitHub, set up your team repo, and install the Python packages your car needs.
 
| | |
|---|---|
| **Hostname** | `burrocar<NN>.local` (NN = your team number) |
| **Starter account** | `burrocar<NN>` (get the password from a lead) |
| **Network** | `IEEE_5G` — your laptop must be on it too |
 
> **Warning:** Never run `sudo apt update` or `sudo apt upgrade`. Newer packages can break your setup. If your Pi breaks, ask a lead to reflash it.
 
### 1a) Log in and create accounts
 
Do this once, from one laptop.
 
1. Put your laptop on `IEEE_5G`, then SSH into the Pi from a terminal:
```
   ssh burrocar<NN>@burrocar<NN>.local
```
2. Create an account for each teammate and give it access to the hardware:
```
   sudo adduser <name>
   sudo usermod -aG sudo,video,i2c,gpio <name>
```
   The `video` group is needed for the camera; `i2c` and `gpio` are needed for the pins. (No spaces after the commas!)
3. Log out with `Ctrl-D`. From now on, everyone uses their own account:
```
   ssh <name>@burrocar<NN>.local
```
 
### 1b) Create the team repo
 
One person does this.
 
4. Fork [`donkey-racers`](https://github.com/TODO/donkey-racers) on GitHub (**Fork** button, top right).
5. Add your teammates and both leads as collaborators (**Settings → Collaborators**).
> Your fork is public. Never commit passwords, API keys, or SSH keys.
 
### 1c) Connect your account to GitHub
 
Everyone does this, on the Pi, in their own account.
 
6. Set your Git identity. Use your GitHub **noreply** email (GitHub **Settings → Emails**) so your personal email isn't public:
```
   git config --global user.name "Your Name"
   git config --global user.email "you@users.noreply.github.com"
```
7. Create an SSH key and print the public half:
```
   ssh-keygen -t ed25519 -C "you@users.noreply.github.com"
   cat ~/.ssh/id_ed25519.pub
```
8. On GitHub, go to **Settings → SSH and GPG keys → New SSH key** and paste the output from step 7.
9. Test the connection. You should see a greeting with your username:
```
   ssh -T git@github.com
```
10. Clone your team's fork into your home directory using the **SSH** link (not HTTPS):
```
    cd ~ && git clone git@github.com:<fork-owner>/donkey-racers.git
```
11. Also clone the fork on your laptop. Repeat steps 6–9 there if you haven't set up Git on your laptop before.
### 1d) Set up your Python environment
 
Everyone does this, on the Pi, in their own account.
 
12. Try installing a package directly:
```
    pip install adafruit-blinka
```
You'll get an `externally-managed-environment` error. Raspberry Pi OS protects the system Python so a bad install can't break the OS. The fix is a **virtual environment** — a private Python install that belongs to you.

13. Create and activate your environment:
```
    python3 -m venv ~/env --system-site-packages
    source ~/env/bin/activate
```
Your prompt should now start with `(env)`. The `--system-site-packages` flag lets your environment see the camera libraries that came with the OS.

14. Install the packages your car needs (this takes a few minutes):
```
    pip install "donkeycar[pi]" adafruit-blinka adafruit-circuitpython-pca9685
```
15. Make the environment activate automatically every time you log in:
```
    echo 'source ~/env/bin/activate' >> ~/.bashrc
```
16. Verify:
```
    python -c "import donkeycar, board; print('ok')"
```
 
### Check your work
 
Each person, in turn:
 
1. SSH into the Pi and `cd ~/donkey-racers`.
2. Create `<your-name>.txt` with a message in it.
3. `git add`, `git commit`, and `git push`. If the push is rejected, `git pull` first, then push again.
When everyone has pushed, `git pull` should show every teammate's file — on the Pi and on your laptop.
 
---
 
## Part II: Team Selfie!
 
1. Shut down the Pi (`sudo shutdown now`) and unplug power. **Never plug in the camera while the Pi is powered.**
2. Insert the camera ribbon into the **camera** port (not the display port): lift the black latch, insert the ribbon, push the latch back down. You may need to push fairly hard to seat it.
3. Power the Pi back on and SSH in.
4. Take the selfie. Have one person hold up the camera so everyone is visible:
```
   mkdir -p ~/Pictures && cd ~/Pictures
   rpicam-still -o team_selfie.jpeg
```
   It may take a couple of seconds and print strange-looking output — that's normal.
5. Press `Ctrl-D` to end the SSH session. Don't reconnect for the rest of this part.
6. Copy the image to your laptop. You can't use GitHub (the picture isn't in your repo), so use `scp` from your laptop's terminal:
```
   scp <name>@burrocar<NN>.local:~/Pictures/team_selfie.jpeg .
```
 
Create a Discord group chat with both leads and your whole team. Message us there and we'll send a link to your team's submission folder — pin it, you'll use it all year. Upload your selfie there.
 
> We can tell a Pi-camera photo from a phone photo, so don't fake it — you'll just hit the real camera issues later.
 
---
 
## Part III: Time to be a detect-ive!
 
In lecture, we attached I2C wires to the Pi. Now you'll use them to detect your first I2C peripheral.
 
1. Shut down the Pi (`sudo shutdown now`) and unplug power before wiring.
2. Get an I2C OLED from a lead and connect:
   | Pi | OLED |
   |---|---|
   | GND | GND |
   | **3.3V** (pin 1) | VCC |
   | SDA (pin 3) | SDA |
   | SCL (pin 5) | SCL |
   > **Warning:** Use **3.3V, not 5V**, and never swap power and ground. Either mistake can damage the Pi or the OLED.
3. Power the Pi on, SSH in, and run:
```
   i2cdetect -y 1
```
   You'll see a grid of addresses (we'll explain it next lecture). If `3c` or `3d` appears, your OLED is connected. If you see only `--`, check your wiring.
4. Screenshot the output and upload it to your team's submission folder.
 
---
 
## Conclusion
 
Message both leads to schedule your check-off. We'll go over your work, ask a few questions to check understanding, and award your points. Congratulations!
 
---
 
## Where This Goes
 
The GitHub workflow, SSH setup, and team repo you built here are what every later lab — and the capstone — runs on. When we release a new lab, click **Sync fork** on GitHub, then `git pull` on your Pi. The I2C bus you just used is the same one your motor driver and sensors will use starting in Lab 2.
