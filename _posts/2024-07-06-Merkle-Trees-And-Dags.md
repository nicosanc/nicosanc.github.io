
### MERKLE TREE

Helps to compute the difference between two sets of records
Useful for:
- Git version control
- leaderless database: cassandra, dynamoDB
- blockchain / crypto

![[Pasted image 20240624123947.png]]
Here we can see that if the 3rd node is updated, its hash updates, as well as every parent node linked to that leaf. This allows for fast comparison between trees to find where differences are in the data / files

The end result of this is a [tree structure](https://www.baeldung.com/cs/binary-tree-intro) in which every intermediate node is a hash of its child nodes until the leaf nodes of the tree are actually hashes of the original data. This means every leaf node is a hash of only a small portion of the data.

- Efficiently verifying that a large amount of data is unchanged and finding exactly where any changes may have occurred
- Efficiently proving that a piece of data is present in a larger data set

The leaves are partitions of an original piece of data, and can be split however we want as long as it is reproducible.
Since it uses hash functions, which produces outputs of fixed size, we can easily determine the size of a merkle tree (and it will always be the same). The actual size of the data does not matter since it will be crunched down to blocks of hashes anyway, each of any size of data

### MERKLE DAGS

Directed Acyclic graphs are the natural representation of data hierarchy, where nodes can only point to child nodes. For example,
a `family tree`. In our case, a `MERKLE DAG` represents a hierarchy of data.

To start, you must 
- Encode the leaf nodes of the graph
- Give them a CID

The CID is generated by passing the entire data of the node (can be its name, its content, metadata, etc.) into a cryptographic hash function. <font color ="blue">Note, the CID is not actually embedded in the node's value</font>

Intermediary nodes look a bit different:
- Each of these will also contain a name representing the subdirectory
- The content of a directory node is actually the list of files and directories it embodies, rather than the content of those files
	- These are represented as a list of CIDs
	- each of which links to a child node in the graph
	- The CIDs are actually embedded into these nodes' value

If a change is made at any level, the change is propagated <font color="blue">to each one of the node's ancestors</font>. For example if a leaf node containing an image is slightly photoshopped, its pixel content changes and thus its cryptographic hash changes with it. This produces a <font color="blue">new CID value</font> which is then updated in its parent node, and finally in any ancestor node of its parent node. This means that Merkle DAGs are <font color="blue">strictly bottom-up</font>. This means parent nodes cannot be created until the CIDs of their children can be determined
- Note: The changes are only made to the nodes directly linked to the changed node
	- This means nodes from other branches in the tree remain unaffected since they're not directly dependent on the changed node
Since the crpyto hash function used make it impossible to make a "self-referential" path through the tree. 
- This an important security guarantee against infinite loops and potential DoS

### Merkle DAGs: Verifiability
Because cryptographic strength hash algorithms to create CID, they offer a high degree of verifiability: <font color="blue">An individual who retrieves data using a content address can always compute the CID for themselves to ensure that they got the right file / data</font> 
This offers two major security benefits:
1. Offers permanence: the data behind the content address will never change
2. Protection against malicious manipulation: An adversary cannot trick you into downloading or opening malicious payload without you recognizing that it has a diff CID from your desired file

The CID of each node depends on the CID of each of its children. As such the CID of a root node uniquely identifies not just that node, but the entire DAG of which it's the root. 

##### Any Node Can Be a Root Node
DAGs can be seen as recursive data structures, meaning every DAG can be broken down into smaller DAGs within itself
Benefits:
- Allows you to retrieve a subgraph of the file system or DB without having to retrieve any other files not directly linked to it
	- ensures protection and privacy of all the other data that is stored in the global DAG
- Allows you to insert a subgraph into another DAG since the CID of the root only depends on its descendants not its ancestors
##### Ensuring A Root Node Exists
It is not strictly necessary to have a singular root, since the data can be structured as multiple DAGs with separate roots
- If you want a single root, one can be created by linking all of the roots to one new root that connects all of the data
- If we don't, we can have separate data structures that simply share data 
Note: If there is no singular root, it will be impossible to access / traverse every node of the data.

### Merkle DAGs: Distributability
Merkle DAGs inherit the distributability of CIDs. This means:
1. Anybody who has a DAG is capable of acting as a provider for that DAG
2. When we are retrieving data encoded as DAG, like a directory of files, we can leverage this fact to retrieve all of a node's children in parallel, potentially from a number of different providers?
3. File servers are not limited to centralized data centers, giving the data greater reach (decentralized network??)
4. Because each node in a DAG has its own CID, the DAG it represents can be shared and retrieved independently of any DAG it is itself embedded in
### Merkle DAGs: DeDuplication
Deduplication means avoiding duplicating an entire dataset anytime it is accessed by someone
DAGs efficiently store data by encoding redundant sections as links
If we wanted to delete one portion of a large DAG, and replace it with another, then all of the other subDAGs can be preserved and just linked to by both the new DAG and the old DAG

![[Pasted image 20240624143159.png]]
Since the cats subDAG is shared by both baf 8 and baf11, it can be just be linked by them without having to duplicate it
This redundancy allows DAGs to be much more efficient
For example, if we think of a web browser:
- Location addressed data (URLs) are used to retrieve a web page, and all of its contents must be downloaded everytime the page is accessed
	- Even though many sites may seem like basic variations of each other in terms of themes or templating, they must each be redownloaded everytime
- If content based addressing was used, then a browser could just link to the stored themes and templates without redownloading them, only downloading whatever is <font color="blue">different</font> between them
	- This is the basic idea of <font color="blue">versioning</font> like GIT

Content Addressing allows us to form distributed, decentralized global file systems. You could access an entire dataset without ever having to actually download the entire dataset as long as the dataset persists (people with network access either using or storing the data). This means the dataset can be stored as a collection of data across many computers. Files can be partitioned, and converted into a DAG
