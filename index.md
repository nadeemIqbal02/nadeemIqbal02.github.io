<html>
<body>
<h1>Architecture which Never Fails but Scales</h1>
<p>
In today’s age, no one can imagine if he is unable to book flight or shop online at any given time due to system failure. Though technology brings everything to your fingertips it’s highly volatile as well at the same time because software must run on a hardware which is built using tens of thousands of small electronic circuits. The probability of failure is high because if the probability of one circuit’s failure is 0.0001 and if there are 10K circuits, the probability of failure is much higher (I believe there is no need to prove 😊).

Redundancy is KEY for any computer architecture whether it’s deployed in-house or Cloud. Redundancy means though you need two items to fulfill the need, you keep adding one(s) in case any of these fails.

When browser request to open a webpage, request reaches to the DNS Server and eventually reach to the web server to process the request but if due to hardware failure, the server is down then request will not be served. To avoid this situation and remain “ALWAYS ON”, there should be an additional server(s) so that if one is down, the request could be served by other servers. Well, how DNS server knows that existing mapping is not valid because the server is down, and it should start routing to the new server. But who will inform DNS server(s) that there is new mapping? Maybe someone updates its Domain to IP mapping manually or automated way, but DNS Servers are scattered across the world, it will take some time before all DNS servers update its mapping which means users trying to access your website will not be able to access for an undefined duration.

Is there a way that we don’t need to update DNS Mapping rather it remains static? Yes, that’s where Load Balancer (NLB) come into the picture (NLB could be hardware or software. F5 is one of the famous Load Balancer). Load Balancer is a component which has the intelligence to route traffic to different servers while retaining single IP Address. So, we define our NLB’s IP in DNS and NLB decides where it should route traffic. How NLB knows if any of the servers are down? There are different mechanisms but easiest is heartbeat between NLB and servers. NLB keeps pushing a message to each server and if it doesn’t hear back acknowledgment then it means that the server is down. So, we resolved the problem. Correct? No, what if NLB itself fails. More NLB(s)? Again issue of DNS update? Think…  there is a way

A single server has limited capacity and it can handle the limited number of requests from customers. we need more computing power so, let’s increase the power of the server itself with higher CPU & RAM. It works and now I’m able to handle more customer requests. It’s called VERTICAL SCALING when you add additional power to the server. But, there is the limit of the RAM & CPU plugged to a server and as we go for higher specs it keeps on getting more and more costly. Just imaging 2xRAM of 64 GB is cheaper than 1x128GB. Not a feasible option because you are bound due to hardware limitation & cost how many customers system can serve at a given time. Just imagine Uber, Ali Baba, Amazon handling millions of concurrent requests how it’s possible.

We need to have more servers which could serve more requests at a given time and NLB should distribute the load to all the active servers. As customer base grows, I keep on adding new cheap servers rather than expensive servers. This is called HORIZONTAL SCALING when you add more nodes (servers) and it proportionally increases the capacity.

</p>
<p>As explained above, redundancy is key to any architecture. But why? It's similar to real life case when one goes for a long drive, ensures that there is at least one additional tire available. Also, we understood that Vertical Scaling is costly and have limitations that it could scale up to a certain point because of hardware limitations. 

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