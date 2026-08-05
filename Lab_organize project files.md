### Create main project directory.
- mkdir command to create new directory.
- ls to list out the content of your home directory.
```
mkdir project_lab

ls -ltr
```
### Create subdirectories for project organization.
- Create three directories named docs, reports and images.
- mkdir command give full path of project_lab and directory names.
- Use ls command with the path of project_lab
```
mkdir /home/user/project_lab docs reports images
ls -ltr
```
### Create sample files in the directories.
- use touch command to create empty files.
- Inside the docs directory, create file called notes.txt
- Inside the reports directory, create file called summary.txt
- Inside the images directory, create file called photo.jpg
- Use ls command to see the files.
```
cd /home/user/project_lab/docs/
touch notes.txt

cd /home/user/project_lab/reports/
touch summary.txt

cd /home/user/project_lab/images/
touch photo.jpg
```
### Copy files into correct directories.
- Use cp command to copy notes.txt from docs into reports.
- copy summary.txt from reports into docs
```
cp /home/user/project_lab/docs/notes.txt  /home/user/project_lab/reports/ 

cp /home/user/project_lab/reports/summary.txt /home/user/project_lab/docs/ 
```
### Move files and verify the project structure.
- Rename the file photo.jpg inside the images directory to project_photo.jpg
- run ls -R ~/project_lab to list all files and directories under project_lab
- Use tree command if it available on your system.

```
mv /home/user/project_lab/images/photo.jpg /home/user/project_lab/images/project_photo.jpg

mv /home/user/project_lab/docs/notes.txt  /home/user/project_lab/reports/

ls -R ~/project_lab
```
