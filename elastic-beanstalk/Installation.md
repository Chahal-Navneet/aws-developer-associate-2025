## Installing Elastic Beanstalk using Windows with Installer script
Prerequisite: Python, Git, [virtualenv] (https://virtualenv.pypa.io/en/latest/installation.html)

1. Install virtualenv
'''
python -m pip install --user virtualenv
python -m virtualenv --help
''' 
If the last line says config file is missing then create a virtualenv.ini file:
'''
[virtualenv]
python = python3.11
always-copy = true
'''
Export the env var:
'''
set VIRTUALENV_CONFIG_FILE=/path/to/my_virtualenv.ini
'''
Add "Scripts" to PATH:
'''
py -m site --user-site // find the folder for Scripts
'''
Open Start Menu → Edit the system environment variables
Go to Environment Variables → Path → Edit → New
Paste the Scripts folder path.
Restart PowerShell/Command Prompt.


2. Clone git repository
'''
git clone https://github.com/aws/aws-elastic-beanstalk-cli-setup.git
'''
3. Run in Command Window or Powershell
'''
python .\aws-elastic-beanstalk-cli-setup\scripts\ebcli_installer.py
'''
4. Verify
'''
eb --version
'''

