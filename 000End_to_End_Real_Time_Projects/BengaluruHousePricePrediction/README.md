# BengaluruHousePricePrediction
### Building a Model to Predict the Price of the House in a Bangaluru
#### Checkout the Data Analysis - https://github.com/reddysrinath16/BengaluruHousePricePrediction/blob/main/model/BengaluruHousePricePrediction.ipynb

## APP OVER VIEW 
-![](TASK.gif)


### Steps
<b>Sslect</b> - Number of Rooms and Number of Bathroom
<b>Select</b> - Area from a particular list of Areas
<b>Click On</b> - Estimated Price in order to predict the price from the EXISTING Data Available(Trained Model)
#### Problem Statement:
    What are the things that a potential home buyer considers before purchasing a house? The location, the size of the property, vicinity to offices, schools, 
    parks, restaurants, hospitals or the stereotypical white picket fence? What about the most important factor — the price?Buying a home, especially in a city
    like Bengaluru, is a tricky choice.With its help millennial crowd, vibrant culture, great climate and a slew of job opportunities, it is difficult to ascertain
    the price of a house in Bengaluru
    
   
      

### Data Information :
* area_type	,availability	,location	,size	,society	,total_sqft	,bath ,	balcony	,price
* Rows*Columns (13320, 9)

### Data Cleaning(Not done much) and dropping some feature
* We have to predict the price so we have to drop this feature(from my point of view in order to make more genralize data)'area_type','society','balcony','availability(as it doesnt matter becox we are predicting price)' 
* Some least number of null Values ao i have literally dropped the values

### Feature Engineering 
* Added new feature(integer) column for bhk (Bedrooms Hall Kitchen form i.e 2 BHK or 2 Bedroom etc) that can actually makes sense to Data
* Total sqft was in ranges and some in other formats so made so transformation and coverted to price per square feet

### Outlier Removal Using Business Logic
* Considering(i am a domain expert) in my case  i.e. 2 bhk apartment is minimum 600 sqft. If you have for example 400 sqft apartment with 2 bhk than that seems suspicious and can be removed as an outlier. We will remove such outliers by keeping our minimum thresold per bhk to be 300 sqft
* We can remove those 2 BHK apartments whose price_per_sqft is less than mean price_per_sqft of 1 BHK apartment
* And where there are more bathroom than bedrom

### Dimensionality Reduction Technique 
    Locations which is a categorical variable. We need to apply <b>Dimensionality Reduction Technique</b> here to reduce number of locations we have used One Hot Encoding For Location
 
#### Model Building ,HyperParamter Tuning and Training
* Used K Fold cross validation to measure accuracy of our LinearRegression model
* We can see that in 5 iterations we get a score above <b><i>80%</i></b> all the time. This is pretty good but we want to test few other algorithms for regression to see if we can get even better score. We will use GridSearchCV for this purpose
* Finding best model using <b>GridSearchCV</b> using various hyperparameters on <b>LinearRegression ,Lasso and DecisionTreeRegressor</b>
* LinearRegression with score a 0.847796
* Lasso with score a 0.726738	
* DecisionTreeRegressor	with score a 0.716064	

# ❤ LinearRegression is our finalized model and exported it in .pickle file for further Our Further Flask app development ❤


### APP development(We are Using the concept of Cross proxy Server for our app)
# Deploy this app to cloud (AWS EC2)

1. Create EC2 instance using amazon console, also in security group add a rule to allow HTTP incoming traffic
2. Now connect to your instance using a command like this,
```
ssh -i "C:\Users\Viral\.ssh\Banglore.pem" ubuntu@ec2-3-133-88-210.us-east-2.compute.amazonaws.com
```
3. nginx setup
   1. Install nginx on EC2 instance using these commands,
   ```
   sudo apt-get update
   sudo apt-get install nginx
   ```
   2. Above will install nginx as well as run it. Check status of nginx using
   ```
   sudo service nginx status
   ```
   3. Here are the commands to start/stop/restart nginx
   ```
   sudo service nginx start
   sudo service nginx stop
   sudo service nginx restart
   ```
   4. Now when you load cloud url in browser you will see a message saying "welcome to nginx" This means your nginx is setup and running.
4. Now you need to copy all your code to EC2 instance. You can do this either using git or copy files using winscp. We will use winscp. You can download winscp from here: https://winscp.net/eng/download.php
5. Once you connect to EC2 instance from winscp (instruction in a youtube video), you can now copy all code files into /home/ubuntu/ folder. The full path of your root folder is now: **/home/ubuntu/BangloreHomePrices**
6.  After copying code on EC2 server now we can point nginx to load our property website by default. For below steps,
    1. Create this file /etc/nginx/sites-available/bhp.conf. The file content looks like this,
    ```
    server {
	    listen 80;
            server_name bhp;
            root /home/ubuntu/BangloreHomePrices/client;
            index app.html;
            location /api/ {
                 rewrite ^/api(.*) $1 break;
                 proxy_pass http://127.0.0.1:5000;
            }
    }
    ```
    2. Create symlink for this file in /etc/nginx/sites-enabled by running this command,
    ```
    sudo ln -v -s /etc/nginx/sites-available/bhp.conf
    ```
    3. Remove symlink for default file in /etc/nginx/sites-enabled directory,
    ```
    sudo unlink default
    ```
    4. Restart nginx,
    ```
    sudo service nginx restart
    ```
7. Now install python packages and start flask server
```
sudo apt-get install python3-pip
sudo pip3 install -r /home/ubuntu/BangloreHomePrices/server/requirements.txt
python3 /home/ubuntu/BangloreHomePrices/client/server.py
```
Running last command above will prompt that server is running on port 5000.
8. Now just load your cloud url in browser (i have been facing some issues😰due to high usage of s3 bucket) and this will be fully functional website running in production ☁️cloud environment😄





