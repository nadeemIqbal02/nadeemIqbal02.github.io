<html>
<body>
<h1>Architecture which Never Fails but Scales</h1>
<p>In the last post, I explained that redundancy is key to any architecture. But why? It's similar to real life case when one goes for a long drive, ensures that there is at least one additional tire available. Also, we understood that Vertical Scaling is costly and have limitations that it could scale up to a certain point because of hardware limitations. 

Hence, for a scalable architecture, it must allow Horizontal Scaling so that adding additional nodes linearly increases the throughput of the system. But horizontal scaling (Partitioning) brings in its own challenges such as;

How to maintain session across multiple servers?

Typical web applications store session information in memory but if the session is stored in node1 and next request goes to node2 where the session is not available, the user will be forced to log in again. Moreover, one server can handle the limited number of requests.

The solution could be to store the session state on a dedicated server (State Server) and all nodes access State Server to validate session state. Yes, it works perfectly but what if state server itself fails then all users are out. Hence, we should have redundancy means more than one state servers. But how state information will be synced between multiple servers? One way could be all web servers read and write to single state server and it replicates this information to other server(s) so that if primary state server fails, all nodes start accessing the alternate server ( need to have some logic to know if primary state server failed then start using alternate ). Works perfectly

But if the state of all users is stored on one server then we stuck into the same problem of scaling which means there is limited memory hence limited state can be stored. In order to have scaling, we should have multiple state servers available at any given time and state of 1 - m users is stored at State Server 1, m - n is stored on State Server 2 and so on. Question? how web server knows that on which state server it should store the state. There are different ways but the easiest one is to use (UserID) % (number of State Servers) =  Server on which State of this particular user to be stored. 

With the aforementioned approach, we are able to scale but what if one of the state servers fails? we should then have back up the server which gets replicated from the primary node. Should we have back up server for each of the state servers? isn't it too much additional cost? Leaving to the readers how we can optimize.

Some would have thought that why don't we use Database to store state but it has the same problem if it fails. Also, Disk operations are low performant (probably subject of another post) 

Consistency Availability Partition (CAP) THEOREM

We observed that one server can handle a limited number of requests and requests should be distributed across multiple servers which means that user logs in to server 1 but next request can go to server 3 hence server 3 should already have the information stored on server 1 before it could process the request. It means that both servers state should be consistent. As requests can come in milliseconds and server 3 might not have updated information.

Just imagine that user added two products to his shopping cart which is stored on server 1 and next request goes to server 3 which don't have this information at the time request comes in which means his shopping cart will be empty. Huge Problem? It means that we should ensure that when the user adds a product to shopping cart, we must ensure that this information is updated to all nodes before confirming user that your products are added to the cart in order to ensure CONSISTENCY. But, during that time when all nodes are being synced, the system is not responsive means not AVAILABLE. In order to ensure CONSISTENCY we had to compromise AVAILABILITY but if we don't wait for data to be synced among all nodes and responds as soon as data is stored in one node means we are compromising CONSISTENCY over AVAILABILITY.

Having said that Partition ( horizontal Scaling ) is mandatory for SCALABILITY which we can't compromise. There is a tradeoff between CONSISTENCY and AVAILABILITY. Based on critically of the system, AVAILABILITY or CONSISTENCY will be compromised. This is called CAP Theorem.

If an architecture could be designed which have all three key factors CONSISTENCY, AVAILABILITY, and PARTITION, it would be a breakthrough in the industry and CLOUD INFRASTRUCTURE. </p>
</body>
</html>