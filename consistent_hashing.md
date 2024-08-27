# Consistent hashing

https://www.toptal.com/big-data/consistent-hashing 

Consistent hashing is an hashing algorithm used to resize/rebalance the data in hash table or evenly distributes the data across the nodes in the distributed system with minimal data movement.

Why do we need hashing algorithm techniques ?

Currently scale which modern applications are operating we cant make it work with one machine as we will face memory, performance etc limitations. So for providing high scale, perfomances we need to choose distributed cloud computing systems.

In distributed systems we partition the data and store the subset of datas in different nodes(Instances/machines).

Normally we can distribute the data by below logic 

NodeID = (hash(data)) / N → Where is the number of node servers 

But if we want to add a new node to meet the scaling requirements or if a node crashed then we need to rebalance the data to N-1 or N+1 nodes , But if this N changes then all the existing data present in the nodes has to rebalance again. So there will be large data moment between nodes.

So we need an hashing algorithm which will meet below requirements.

- Data should be distributed as evenly as possible
- Distribution should be independent of number of servers

Consistency hashing solves above problems using below technique 

We will distributes all the data points and nodes across a circle with 0 to 360 degree radius where each key and nodes falls into certain degree angle, And we are going to choose the nearest node for that data key on counter clockwise direction. 

Suppose if an exist node crashes or new node gets added we will distribute only the keys present in between to the next node and previous node. So remaining keys present in between the other nodes keeps idle.

Also to distribute the data evenly we create different labels for same node like Node A contains A1, A2 , A3 labels which will assigned to certain angle , So keys will distributed in between, Also here we can give more weight to certain machines.

Also we call these different labels as virtual nodes which represent one true node and helps in distributing data evenly across the cluster.

In general, only `k/N` keys need to be remapped when `k` is the number of keys and `N` is the number of servers (more specifically, the maximum of the initial and final number of servers).

<img width="1091" alt="Screenshot 2024-08-27 at 5 57 37 PM" src="https://github.com/user-attachments/assets/46ba9671-869b-4e29-af00-c2e08eab48cb">
