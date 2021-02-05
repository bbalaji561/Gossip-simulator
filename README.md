# Gossip-simulator
Gossip type algorithms can be used both for group communication and for aggregate computation. The goal of this project is to determine the convergence of such algorithms through a simulator based on actors written in F#. Since actors in F# are fully asynchronous, the particular type of Gossip implemented is the so called Asynchronous Gossip.

To calculate the convergence of gossip algorithms: gossip and push sum, when used for either group communication (gossip algorithm) or while computing the aggregation (push sum algorithm) using simulation models.
Given is one amongst the different network topologies (Full, 2D, Imp2D and Line) with a number of nodes as size of each network.
Our goal is to experiment the given network of n nodes using akka actors and find the convergence time.

### Interesting Points / Findings:
### Gossip:
1. First we implemented the model where we assume that the convergence occurs or program terminates when all the actors were terminated.
    - When all the actors were continuously sending messages till they terminate, we have noticed that, in a few cases when all actors except one or two were terminated, the leftover actors just keep sending messages and never terminate because they don’t get messages from any other actor.
    - To handle this, we have implemented a method where the actor keeps track of the number of its neighbours that are not terminated, and once all of its neighbours were terminated, this actor also terminates itself. The reason behind this approach is if none of its neighbours are active, the actor is not going to spread the rumour further and so logically terminates itself.
2. Second model is implemented in a way that it terminates when all of the actors have reached the rumour at least once.
    - To avoid the issue in the above model we have built this approach so that the system terminates once all of them have received the rumour.
### PushSum:
For pushsum we assume that convergence occurs when the nodes’ ratio has not changed more than 10^-10 in three consecutive rounds and the program terminates when all actors are terminated.

​Note: ​When we implemented this algorithm as mentioned in the given document(which is actor sends messages upon receive), it became sequential and the program halted after one of the nodes converged and it didn't proceed further.

To address the above issue we handled as below but we also have observed that all the actors were not converging when all the actors were sending messages to neighbours and when all of them were terminated except few. In such cases, the system runs into a loop with few actors continuously sending messages without terminating or the actors stopped transmitting and other actors are not receiving any messages.
 
We handled the above problem similar to the gossip model by tracking the active neighbours. We tried two approaches here with one model sending randomly to neighbours and the other sending messages only to the active neighbours. In the first case, we have noticed a few times that systems did not terminate in full and line topology because of the random functionality and in such situations the actor sends the messages to only the active neighbours to help them converge.

In the above two approaches, the second model showed better performance and also the ratios as this model only sends messages to active neighbours as sending them to already terminated nodes is not going to achieve anything as these messages are simply ignored by the terminated node.

​Note: ​To continuously send the messages, the actor sends a signal to itself after each second everytime using the Akka Dispatchers. This dispatcher is set to send a self message to the actor after one second to keep it continuously sending the messages. Also, in the dispatch wait time one sec other actors were picked up and run thus concurrently running all the actors.

### Input:
1. numNodes - Number of nodes(actors) in participation, 1000 always in this case
2. topology - One of the networks: full, 2D, imp2D and line
3. algorithm - Either gossip or push-sum
Additional argument for project2 Bonus
4. removeNodes - Remove the nodes from the participation List.

### Commands to run:
#### For project2:
dotnet fsi --langversion:preview project2.fsx <numNodes> <topology> <algorithm>
#### For project 2 Bonus:
dotnet fsi --langversion:preview project2Bonus.fsx 1000 <topology> <algorithm> <remNodes>

### Implementation: Project 2
### Initial Steps
1. Built the topology by constructing neighbors list for each actor during its initiation. The neighbors list consists of only the remaining nodes present in the network.
2. Now, will start the given algorithm by randomly selecting a single actor/participant.
### Algorithm
3. The information of the actor’s neighbors are stored as a list (neighborsList) which is constructed during its initiation    
4. Upon external rumour message arrival (from one of its neighbors actors), the actor sends the rumour message to one of its neighbours and also maintains its own state (external message count) of tracking the number of times rumour message is received by incrementing it by 1.
5. Also at a periodic interval it sends the rumour message to itself (self message) from the time the first message to it is received until it terminates.
### Termination
6. The program terminates when all of its actors receive the rumour message exhaustion number of times.
### Output
7. Time algorithm takes to converge for the given topology with n nodes in milliseconds

### Project 2 - Bonus
For the Bonus, we developed an algorithm based on the number of nodes that the main process invokes (removing failure nodes, only remaining nodes is present) and handling convergence issues.

Implemented the failure model by deleting randomly the number of nodes which is provided by the user as one of the inputs. Observed how much time the remaining nodes have taken to converge.

### Detailed Findings:
1. Convergence was noticed for the remaining nodes present in the 2D, Imperfect 2D and Full network topologies.
2. But, it failed to converge for Line topology. For instance, we had initially taken 200 nodes and removed 1 node. Once, the random node that was picked to be deleted was 100, it is observed that 1-99 had converged and the remaining didn’t as it did not receive any gossip message.
3. Even Killing more than 10% of the nodes, the line topology didn’t converge
4. Full, 2D, Imperfect 2D topologies handled the node failures correctly and converged even when the number of nodes killed were extremely large.
5. When the number of failure nodes percentage increased above 50%, we observed that the algorithm did not converge.
6. Imp2D was converging the fastest then followed by 2D and then Full.( imp2D < 2D < Full )

### Initial Steps
1. Built the topology by constructing neighbors list for each actor during its initiation. The neighbors list consists of only the remaining nodes present in the network.
2. Now, will start the given algorithm by randomly selecting a single actor/participant.
Algorithm
3. The information of the actor’s neighbors are stored as a list (neighborsList) which is constructed during its initiation
4. Upon external rumour message arrival (from one of its neighbors actors), the actor sends the rumour message to one of its neighbours and also maintains its own state (external message count) of tracking the number of times rumour message is received by incrementing it by 1.
5. Also at a periodic interval it sends the rumour message to itself (self message) from the time the first message to it is received until it terminates.
### Termination
6. The program terminates when all of its actors receive the rumour message exhaustion number of times.
### Output
7. Time algorithm takes to converge for the given topology with n nodes in milliseconds
