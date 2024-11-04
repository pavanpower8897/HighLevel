Objects are massive unstructed data. Its basically chunk of data like images/videos and VMs.

Famous Object stores are S3 AWS
In AWS S3 by looking at the file path of the object it looks like objects are stored in file storage but its just an illusion inside there will be a list of buckets and each bucket will be allocated to a unique identifier accross all the accounts and inside each bucket will contain a list of objects which will be associated with list of objects with unique ids.

Why cant we use Databases to store objects ?



Why cant we use file systems to store these objects ?


<img width="1731" alt="Screenshot 2024-09-28 at 5 13 00 PM" src="https://github.com/user-attachments/assets/b447a5a0-d762-4b7d-9aa9-1e6803bee274">


<img width="1792" alt="Screenshot 2024-09-28 at 5 13 56 PM" src="https://github.com/user-attachments/assets/d08c12c2-6169-44a1-9cc1-5e072c38bf77">


Erasure Coding: 
Erasure coding is a method used in data storage to provide redundancy and fault tolerance while minimising storage overhead.
In erasure coding data will be divided into blocks and redudant blocks are generated with mathematical computation between blocks, Now these redundant blocks we call it as parity blocks 
using this we can reconstruct the orgiinal block,
So using erasure coding we can provide same level or redundancy/availability/fault tolerance with minuimum storage overhead 

![WhatsApp Image 2024-11-05 at 1 22 48 AM](https://github.com/user-attachments/assets/d1e9340f-44ea-4470-add8-3c8a3c8009a4)
![WhatsApp Image 2024-11-05 at 1 22 49 AM (1)](https://github.com/user-attachments/assets/8f33b1ed-6dd4-447a-84bd-b3b8a907f4e9)
![WhatsApp Image 2024-11-05 at 1 22 49 AM](https://github.com/user-attachments/assets/46e505f5-9322-40c5-94fc-529d488625b9)
![WhatsApp Image 2024-11-05 at 1 22 50 AM](https://github.com/user-attachments/assets/8e2dffdc-00bb-4deb-9bb0-029214dfa849)
