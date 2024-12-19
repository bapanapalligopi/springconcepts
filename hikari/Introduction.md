whenever we are going to fetch some data form the database and execute some operation to the database we need a active connection to the database


with out eestablish a connection we are not able to access the database, and we need to disconnect to the database

this opening a connection and disconnect a connction to the dtaabase is decrease the application performance

![image](https://github.com/user-attachments/assets/2606553d-abbd-49ae-baea-5d8129b75c37)

the connection pool

a connection pool is a caching of database connection maintained by o that connection can bereused
improves the performance
resuing existng connection rather than creating a new connection for e very request
![image](https://github.com/user-attachments/assets/6cade7c9-7457-4c8a-a5f7-1b82fbacfed2)

reduce the number of new connection

 a connection is a active connection

![image](https://github.com/user-attachments/assets/ac791092-e06f-4f35-b065-c2828b03ae8b)
![image](https://github.com/user-attachments/assets/b580bd14-cc2a-45db-a1b0-ccd2fb52cfe7)

springboot has the default connection pool management called hikarCP
hikariCP is a jdbc data source implementation, that provides a cp, light weight and best performance

dependecies
come automatically with starter-data-jpa and jdbc
