# 🧒 4th Grade Simple SQL DBA Lab Setup Guide

**What this guide will teach you:** How to set up your own little computer laboratory to learn database skills. Think of it like building your own play kitchen, but for learning databases!

---

## 📚 What You Will Learn

By the end of this guide, you will have:
- ✅ A fake computer in the cloud (Azure) that you can practice on
- ✅ A pretend server on your own computer (like a sandbox)
- ✅ SQL Server Management Studio (SSMS) - this is like your database remote control
- ✅ Everything you need to start learning database skills safely

**Time needed:** About 2-3 hours to complete everything
**Cost:** Free!

---

## 🧰 What You Need First

Before we start, make sure you have these things on your computer:

### Your Computer Needs:
- **At least 8 Gigabytes (GB) of RAM** - This is like your computer's short-term memory
- **At least 20 GB of free disk space** - This is like a closet where your files live
- **Good internet connection** - We'll be downloading stuff from the internet

### What You'll Need to Create:
- **An email address** - You need this to sign up for Azure
- **A new email address** - If you've used Azure before, you might need a fresh one

**How to check if your computer is strong enough:**
1. On Windows: Right-click "This PC" → Properties
2. Look for "Installed memory (RAM)" - should say 8GB or more
3. Right-click "This PC" → Properties → Local Disk (C:) - should show lots of GB free

---

## 🏠 Option 1: The Easy Cloud Setup (Recommended)

**What this is:** Think of this like getting a free trial of a toy store where you can practice with all the toys without breaking anything!

### Step 1: Get a Free Cloud Account

**What you're doing:** Creating a pretend internet computer that Microsoft will let you use for free.

**How to do it:**
1. **Open your web browser** (Chrome, Firefox, Edge - whatever you use)
2. **Go to:** `portal.azure.com`
3. **Click the blue "Start free" button**
4. **Sign up with a NEW email address** (If you used Azure before, use a different email!)
5. **Use a Gmail trick:** Add `+sqldba` to your email like: `yoursname+sqldba@gmail.com`
6. **Fill out all the required information**
7. **Wait for Microsoft to call you** or **text your phone number** to confirm it's really you

**Why you need a new email:** Azure remembers if you've had a free trial before. The trick makes them think it's a new person!

### Step 2: Create Your First Cloud Database

**What you're doing:** Building your first pretend internet computer with database software.

**Step-by-step:**
1. **Click the "+" button** (plus sign) in the top-left corner
2. **Type "SQL"** in the search box
3. **Click "SQL Database"**
4. **Look for a yellow banner that says "Want to try Azure SQL Database for free?"**
5. **Click "Apply offer"** on that banner
6. **Fill out the form:**
   - **Subscription:** Choose "Free Trial" (it should be there automatically)
   - **Resource group:** Type `SQLDBA-Lab-RG`
   - **Database name:** Type `MyFirstDatabase`
   - **Server name:** Type `myserver-1234` (replace 1234 with your birth year)
   - **Password:** Make a strong password (we'll need this later)
   - **Location:** Choose "East US" (or whatever is closest to you)

7. **Click "Review + create"**
8. **Wait for the green checkmark** that says "Validation passed"
9. **Click "Create"**
10. **Wait for a big blue box that says "Deployment in progress"** - this takes about 2-3 minutes

**What you just created:** A computer in the internet that can store and manage databases. It's like having your own virtual filing cabinet!

### Step 3: Connect to Your Cloud Database

**What you're doing:** Learning how to talk to your new internet computer.

**How to do it:**
1. **Go back to:** `portal.azure.com`
2. **Look for your new database** (should be in the list on the left)
3. **Click on your database name**
4. **Click "Query editor"** on the left side
5. **Click "Use SQL authentication"**
6. **Type your server name** (what you named it in Step 2)
7. **Type your password**
8. **Click "OK"**

**Test it works:**
1. **Type this in the big box:** `SELECT @@VERSION`
2. **Click the blue "Run" button**
3. **You should see text about SQL Server appear** - that's your new database computer saying "Hello!"

**🎉 Congratulations!** You now have your first cloud database!

---

## 💻 Option 2: The Local Computer Setup

**What this is:** Installing database software directly on your own computer, like installing a video game.

### Step 1: Download SQL Server Management Studio (SSMS)

**What this is:** This is your remote control for databases. Think of it like a TV remote, but for databases!

**How to download:**
1. **Open your web browser**
2. **Go to:** `learn.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms`
3. **Click the blue "Download SSMS" button**
4. **Wait for a file to download** (it will appear at the bottom of your browser or in your Downloads folder)
5. **Double-click the downloaded file**
6. **Click "Yes"** if Windows asks "Do you want to allow this app to make changes to your device?"
7. **Click "Install"**
8. **Wait for it to finish** (this takes about 5-10 minutes)
9. **Click "Restart"** if it asks you to restart your computer

**What you just downloaded:** A program that lets you connect to and manage databases, like a universal remote for different types of databases!

### Step 2: Download and Install SQL Server (The Database)

**What you're doing:** Installing the actual database software on your computer.

**How to do it:**
1. **Go to:** `www.microsoft.com/sql-server/sql-server-downloads`
2. **Click "Developer"** (it's free, not the trial!)
3. **Click "Basic"** installation (this is the easiest)
4. **Wait for the file to download**
5. **Double-click the downloaded file**
6. **Click "Yes"** when Windows asks permission
7. **Click "Install SQL Server"**
8. **Accept the license agreement**
9. **Click "Install"**
10. **Wait for installation to complete** (this takes 10-15 minutes)
11. **When done, click "Close"**

**What you just installed:** The actual database software that stores and manages your data. Think of it like installing Microsoft Word - now you can write and save documents!

### Step 3: Connect SSMS to Your Local SQL Server

**What you're doing:** Learning how to use your new remote control.

**How to do it:**
1. **Click the Windows Start button**
2. **Type "SSMS"** and press Enter
3. **In the "Server name" box, type:** `localhost`
4. **In the "Authentication" box, choose:** `Windows Authentication`
5. **Click "Connect"**

**Test it works:**
1. **In the left panel, click the plus "+" next to "Databases"**
2. **You should see system databases like "master", "model", "tempdb"**

**🎉 Congratulations!** You now have SQL Server running on your own computer!

---

## 🐳 Option 3: The Modern Container Setup (For People Who Like Speed)

**What this is:** Using a modern way to run database software in a little box, like having a tiny computer inside your computer.

### Step 1: Download and Install Docker

**What this is:** Docker is a program that lets you run other programs in small, safe containers.

**How to do it:**
1. **Go to:** `www.docker.com/products/docker-desktop`
2. **Click "Download for Windows"**
3. **Double-click the downloaded file**
4. **Click "Yes"** when Windows asks permission
5. **Click "Ok"** to install required components
6. **Click "Close and restart"** when done
7. **Restart your computer**

**What you just installed:** A program that creates tiny, isolated environments for running software safely.

### Step 2: Download and Run SQL Server Container

**What you're doing:** Starting up a database inside a safe little box.

**How to do it:**
1. **Click the Windows Start button**
2. **Type "PowerShell"**
3. **Right-click "Windows PowerShell" and choose "Run as administrator"**
4. **Click "Yes"** when Windows asks for permission
5. **Type this command exactly (copy and paste):**
   ```powershell
   docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=YourStrongPassword123!" -p 1433:1433 --name sql1 --hostname sql1 -d mcr.microsoft.com/mssql/server:2022-latest
   ```
6. **Press Enter**
7. **Wait for it to download and start** (this takes 5-10 minutes the first time)

**Important:** Replace `YourStrongPassword123!` with a real password you can remember!

### Step 3: Connect SSMS to Your Container

**How to do it:**
1. **Open SSMS** (if you haven't installed it yet, do Option 2 Step 1 first)
2. **In the "Server name" box, type:** `localhost,1433`
3. **Choose "SQL Server Authentication"**
4. **Type "sa"** as the login name
5. **Type your password** (the one you used in the Docker command)
6. **Click "Connect"**

**🎉 Congratulations!** You now have SQL Server running in a modern container!

---

## 🔍 How to Use SSMS (Your Database Remote Control)

**What this is:** Learning how to use your new tool, like learning how to use a remote control.

### Basic Navigation:

**The left panel is your tree of things:**
- **Databases:** Like different filing cabinets
- **Server Objects:** Things that help the database work
- **Security:** Who can access what
- **Management:** How to keep everything healthy

**Try these simple commands:**

1. **Create a new database:**
   - Right-click "Databases" → "New Database"
   - Type "TestDatabase" as the name
   - Click "OK"

2. **Run a simple query:**
   - Click "New Query" button (top toolbar)
   - Type: `SELECT GETDATE()`
   - Press F5 or click the green "Execute" button
   - You should see the current date and time

3. **Create a test table:**
   ```sql
   CREATE TABLE MyFirstTable (
       ID INT PRIMARY KEY,
       Name VARCHAR(50),
       Age INT
   )
   ```

4. **Add some data:**
   ```sql
   INSERT INTO MyFirstTable (ID, Name, Age)
   VALUES (1, 'John', 25), (2, 'Mary', 30), (3, 'Bob', 35)
   ```

5. **See your data:**
   ```sql
   SELECT * FROM MyFirstTable
   ```

---

## ✅ Testing Your Setup

**Before you continue to the main training program, make sure everything works:**

### Check List 1 - Azure Setup:
- [ ] You can log into Azure portal
- [ ] You can see your database
- [ ] You can run a query in the Query Editor
- [ ] You see SQL Server version information

### Check List 2 - Local Setup:
- [ ] You can open SSMS
- [ ] You can connect to your local SQL Server
- [ ] You see system databases in the left panel
- [ ] You can create a test database

### Check List 3 - Container Setup:
- [ ] Docker is running (you see the whale icon in your system tray)
- [ ] You can run `docker ps` in PowerShell and see your container
- [ ] You can connect with SSMS to localhost,1433
- [ ] You can run the same test queries

### Ultimate Test:
Try this query in any of your setups:
```sql
SELECT 
    @@VERSION as SQLServerVersion,
    GETDATE() as CurrentTime,
    DB_NAME() as CurrentDatabase,
    USER as CurrentUser
```

You should get back information about your database system!

---

## 🚨 Troubleshooting (When Things Go Wrong)

### Problem: "I can't connect to Azure SQL"
**Solution:** 
1. Check your server name (should end with .database.windows.net)
2. Check your password
3. Make sure you're in the "Query editor" section

### Problem: "SSMS won't start"
**Solution:**
1. Right-click SSMS → "Run as administrator"
2. If still won't start, try restarting your computer
3. Download SSMS again from Microsoft

### Problem: "Docker container won't start"
**Solution:**
1. Make sure Docker Desktop is running (check system tray)
2. Try running: `docker stop sql1 && docker rm sql1` then run the start command again
3. Check if port 1433 is already being used

### Problem: "I forgot my password"
**Azure:** You can't change it easily - delete and recreate
**Local/Container:** Create a new login or reset the sa password

---

## 🎯 What You've Accomplished

**Congratulations!** You now have:
- ✅ A real database system to practice with
- ✅ The tools to manage databases
- ✅ Basic understanding of how to connect to databases
- ✅ A safe place to make mistakes and learn

**Next Step:** You're ready to start the 7-day SQL DBA training program! Your lab is set up and you know how to use your tools.

**Remember:** 
- It's okay to make mistakes while learning
- Always practice in your lab first before trying anything on real systems
- If you get stuck, go back to this guide
- The cloud setup is easiest for beginners
- The local setup gives you the most control
- The container setup is the most modern

**Good luck with your database learning journey!** 🚀