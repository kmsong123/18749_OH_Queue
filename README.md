# 18749_OH_QUEUE
18749 BUILDING RELIABLE DISTRIBUTED SYSTEMS


18-749 PROJECT SUMMARY 
TEAM 05 - CMU OHQ


SECTION 1: TEAM MEMBERS

1. Pinak Sawhney

2. Alina Rath

3. Madhurachaitra Manohar

4. Chung Lee

5. Jiahao Zhang

6. John Lennon 

7. Joseph Wang

8. Manini Amin

9. Shreyas Gatuku

10. Kevin Song

11. Yogesh Narayan Vaithiyanathan


CONTRIBUTIONS

The entire project was divided into 5 phases and the team was split into 4 groups of two members each and one group of three members.

Replication Manager : Alina and Madhura

GFD: Kevin and John

LFD: Shreyas and Manini

Clients: Pinak and Jiahao

Replica: Joseph, Chung and Yogesh


SECTION 2: PROJECT IMPLEMENTATION

IMPLEMENTATION DECISIONS

1. TOTAL ORDERING. 

In active replication, total order was done by the process of majority voting. The replication manager would inform all replicas of all membership changes along with the IP addresses of all alive replicas. If there exists at least 2 alive replicas, then with every subsequent message from the client, the replicas talk to each and share their votes. Based on majority of votes collected, the states are updated. If there is a tie, both the votes are updated in a sequential order. Thus, total order between systems is maintained via majority voting and replicas are consistent throughout the entire period of time.

2. DUPLICATE DETECTION.

For our application we wanted to mimic an office hours “stack” where the last person (member id) would get serviced first. This meant that the only state needed was a variable keeping track of the last client message. Therefore the only action happening was an idempotent operation and for active replication we wouldn’t need duplicate detection. 

3. CHECKPOINTING. 

Checkpointing is done in both active and passive replication.

In active replication, the alive replica checkpoints its latest state to the replica which has just come up. Changes in membership along with the IPs of all alive replicas are broadcasted by the replication manager to all the replicas. With this information, the replica which has been alive, implements blocking and sends the checkpointed state to the replica which has just come up.

In passive replication, the checkpoints are sent from the primary replica to all the backup replicas at two points. Firstly, when a new backup replica is brought up,  the primary sends over its latest checkpoint to it. After this, the system has a checkpointing frequency, which determines the rate at which the primary updates its latest state to the backup replicas.

4. LOGGING.

Logging is done in both active and passive replication. The alive replicas constantly log data. In active replication, the logged data is transferred from the alive replica to a newly brought up replica. This logged data is to show the order in which the data is to be processed.
In passive replication, the primary and backups log data. The primary decides the order in which data is to be processed, and thus, during checkpointing, the primary replica also sends the logs to its backups to declare the order of message processing.

In both cases, logged data is erased till the point where data has been checkpointed.
 
5. ENSURING QUIESCENCE.

Quiescence is implemented during checkpoint and log transfer from primary to all the backups in passive replication, and from a replica to newly brought up replica in active replication. 
During quiescence, the replicas still keep themselves available to the clients to ensure there is 100% availability, but the replicas don't update their state. All messages received from clients are logged during quiescence and are not processed until unblocking is done.

6. ASSUMPTIONS IN IMPLEMENTATION.

a. Replicas cannot be taken down during quiescence.

b. Replicas cannot be brought up before the LFD has declared them to be dead.

c. Two replicas cannot be brought up at the same point of time.

7. BLOCKING.

We implement blocking only during transfer of checkpoints and logs between replicas. This is done to ensure that there is no state change while we transfer checkpoints and logs. If we do not block, a replica could update its state and then die. But the other replicas would not have received the latest state that was updated, thus introducing an inconsistency in the system.

SECTION 3: REFLECTION

1. VULNERABILITIES.

a. Our system is weakest when the three assumptions are targeted. We would get stuck in infinite loops with no visible state changes when the replicas are brought down during quiescence. The clients are still working and state changes occur in the background, but the foreground visible terminal would be hung. 

b. Consensus protocol has timeouts in receiving votes. Thus, even if a replica goes down, the time out ensures the other replicas detect it and continue processing with the remaining replica.

c. Our system would be slow if it had a replica with thrashing membership.

d. The system is non fault tolerant during quiescence as it assumes that no replica can be taken down at this point of time.

2. FUTURE WORK.

a. Add timeouts during quiescence to support bringing down of replicas during blocking. This removes the issue of terminal hangs and also deals with the non fault tolerant window currently present in our system.

b. Use of locks and synchronisation to avoid minor race conditions present in our working model. 

