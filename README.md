- # **Abstract**
    - We present the Myel network, a novel and fully decentralized protocol for content delivery. We create a new paradigm for content delivery, with economic actors and economic units which differ from traditional content delivery networks (CDNs) in that __users__, rather than content publishers, pay when accessing content from caches. This cache-client economic pair makes Myel an optimal network for providing a caching layer to decentralized protocols.
- # **Introduction**
    - The development of decentralized content delivery network (CDNs) has been an outstanding research problem since the 1990s [1]. Their core premise lies in the promise of personal computers acting as caches to deliver content to other users browsing web-content. 
    - In most of the existing setups, called collaborative cache setups, these decentralized approaches retain an economic structure that is akin to that of a centralized CDN [2]. Content publishers pay users that provide caching services to store their content. Content clients are then able to retrieve the content they need from these caches, effectively creating an base economic unit composed of the publisher-cache-client triad.
    - This in many ways imitates the setup of a centralized CDN where content creators and providers pay CDN companies  like Akamai and Cloudflare to cash their content in large data-centers, which is then delivered on to content clients. The main difference here lies in the agents acting as caches: in the collaborative cache approach, these caches are individuals joining the network that provide their caches as a service. The aim of existing decentralized CDNs is thus to create a distributed and trustless market with which publishers can hire caches on an as-needed basis, without constraining these parties with long-term business commitments [2].
    - Though it may seem natural to mimic existing centralized CDN architectures when building a decentralized CDN, this creator-cache-client triad introduces a host of problems. 
        - These systems typically still have single points of failure. The content creator and provider still needs to host a server which contains a list of caches that are currently holding the content. If the content provider's server goes down, users can no longer access the content. This makes these network particularly vulnerable to censorship. 
        - Secondly these systems introduce new kinds of financial attacks on the network. Content clients and caches can collude to receive payment __even when content has not been transmitted__. Though solutions exist for mitigating these so-called __cache accounting attacks__, the overhead they introduce effectively renders real-time content delivery impossible [3, 4]. 
    - The Myel network addresses these shortcomings by introducing a novel framework for content delivery where __clients__ pay when accessing content.  The base economic unit is now a cache-client pair, which renders attacks where cache and client collude to receive payment from a provider moot. A content publisher deploys content to the Myel network and content is routed through the network of caches in an automated fashion. Content clients then find the content they are looking for, not through a centralized index maintained by the content publisher, but by querying the network of caches directly. Granted, this content discovery introduces additional overhead but it mitigates the risk of a single point of failure, and the risk is instead distributed across the multiple caches that host the content. The risk of censorship and content downtime is lesser in this sort of setup.  
    ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Fmyel%2FBd0rdOTBG5.png?alt=media&token=134be933-1b02-417c-94f1-44c51ba5f379)
    - > Figure 1: Different CDN architectures
    - As such the Myel network creates a CDN-like decentralized network of caches that is both censorship resistant and more resistant to cache accounting attacks. The new economic model, however, where users pay to access content; is not directly translatable to existing centralized CDN systems. As such we postulate that the main application of the Myel network is to serve as a caching layer for decentralized protocols. These are systems in which users are used to storing private keys and paying a decentralized network to execute business logic or interact with a service. (example: Chainlink, Ethereum NFTs).
    - Currently many decentralized applications interface with resource intensive protocols (Ethereum [7] for example). To interact with these protocols, users still need to run protocol 'nodes' which have intensive computation and storage requirements. As a result, third-parties which manage 'gateway nodes' to protocols have flourished. They allow applications to interface with decentralized protocols through a managed API, effectively outsourcing the cost of running these protocol nodes. These API services rely on traditional CDNs to cache and distribute the workload on their servers, similarly to traditional client-server systems. The Myel network can act as an interface for resource intensive protocols, caching API requests to gateway nodes in a decentralized, performant and resilient way. 
- # **Implementation**
    - Users interact with the Myel network via an Exchange interface named Hop. The implementation written in **golang** runs as a background process and is designed to be highly resilient to chaotic network conditions, device shutdowns and other hazards encountered when running on end user devices. A CLI is available to control the daemon and a desktop UI will also be available in the near future.
    - The Hop Exchange requires access to a wallet for storing peer identity private key and keys for payment protocol addresses. The interface is highly modular and can consume keys stored in the Apple/Unix Keychain or in a Ledger. It provides basic methods for withdrawing funds as well.
    - # Content Routing
        - Content routing is a core component of the Myel CDN. It directly impacts how quickly content is delivered to end users. Here we define two key aspects:
        - ### How content is dispatched to cache providers
            - When Myel nodes add content to the network, the content is dispatched to specific groups of cache provider nodes. This new content is advertised to the caches with a gossip messaging protocol. Subnetworks of providers are grouped around gossip topics and clients target relevant caches by publishing to those topics. It is then the cache providers' decision to accept the request and retrieve the content from the client to serve it for future requests. Note that subnetworks are primarily created around geographic regions, but can also be around communities who share interest for similar content.
                 ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Fmyel%2FK54Ii5dYFJ.png?alt=media&token=303983d7-b467-4e9e-a8b0-615f9eaca0b2)
                - > Figure 2: Clients target relevant caches by publishing new content to a relevant gossip topic. This content is then hosted by subnetworks of providers associated with that topic. 
                - > This system has been implemented and is currently being tested.
                - ```go
                   // Content is dispatched with a simple call
                    err := exchange.Dispatch(c cid.Cid)
                    ```
                - Further, we introduce an automated mechanism for providers to select which content they should serve. This mechanism allows for the network to regulate the lifecycle of content on the network and grow the number of providers hosting a piece of content to meet increases in demand. Underpinning this are two principal mechanisms:
                - On top of the initial cache dispatch, providers can ask each other for their retrieval logs in order to refresh their supply with popular content. 
                - This background job routine can also be enhanced with a probabilistic model to determine the best provider-content pairing. Local nodes continuously keep track of the content they serve and the subsequent rewards received for that content. A lightweight probabilistic model, such as logistic regression or a random forest, can then infer what kinds of content (for example static pages versus videos) are most profitable for the node to host. 
                    - > This still needs to be proven, our thesis is that it will greatly optimize the performance of the network.
        - ### How content is discovered by clients 
            - Fast and efficient strategies for content discovery are critical to the functioning of the Myel network. This is the process by which a client is able to find a cache that is hosting the content they are requesting. Unlike a traditional CDN or a collaborative cache CDN, which use centralized indices of content-cache pairs to route content to a client, the Myel network needs a content discovery system by which a client can quickly search the network for the they are looking for, without relying on a single centralized index. 
            - Traditionally, a HTTP request connects to a DNS which then returns the address of a CDN point of presence (POP), if the POP has the content cached it returns immediately or asks the origin server for it, caches it and finally returns it. This is very simple and performant but assumes we can trust the DNS provider, the CDN provider and the origin server. If any part of the system fails or the link is corrupted users have no alternatives to access their content.
            - Myel uses interplanetary linked data structures (IPLD) and its immutable content identifiers (CID) to verify the integrity of content and declare encoding and hashing functions explicitly. In this data structure, content is chunked and split into different sized blocks forming a directed acyclic graphs (DAG). IPLD makes the system interoperable and resilient whilst remaining flexible such that it can be used for any kind of data model.
            - Developers can use the naming system of their choice (e.g. IPNS, ENS, etc.) to point to a CID representation of the content. Any time the content is mutated, the naming record should be updated with a new CID. In addition, IPLD selector specs are available to declare which parts of the content graph to query. This can be represented via a simple path notation, for instance: 
            - `<CID>/images/Frog.jpeg` 
            - or a more complex serialized declaration for traversing a graph and returning specific nodes. By default, queries for a root CID fetch the entire graph. 
            - Myel offers multiple mechanisms for content discovery based on the use case. The first, gossip-based, is similar to Gnutella and fully decentralized though may have a higher latency, the second Hub-based offers more performance with a compromise on decentralization.
                - **Gossip System:** Content discovery begins by publishing a gossip message containing a CID and selector to one of the provider subnetworks. The gossip message is propagated across the network of providers who check their supply and decide if they want to reply with an offer or simply ignore the message. To send the response, each peer recursively forwards the message back to the publisher. This adds more privacy as it becomes harder to trace a query back to a client as the network grows. The offer contains terms under which a provider is willing to accept a data pull request.
                    - Depending on the client preference (i.e. faster or cheaper transfer), an offer is selected and the client creates a new deal and opens a data pull request with the provider. The provider validates the deal and if it is compatible with the offer will initiate the payment protocol.
            - ```go
                // Start by creating a new session for a root cid
                session := exchange.Session(context, cid)
                // Query the network for the best offer
                session.QueryGossip(context)
                // Start the transfer when we have a satisfying offer
                session.StartTransfer(context)
                // Wait for the transfer to complete
                error = <- session.Done():
                // if err == nil the transfer was successful and content is in our blockstore
            - **Hub System**: Another system for discovery on the Myel network, in the vein of Gnutella2 [5], is to create a special class of cache called Hub nodes, which index content-caches pairing in their local area. Whenever a cache receives new content, they inform their nearest hub to update its index. Whenever a client wants to access content they then query their nearest Hub for the cache address that hosts the desired content. Hubs can also cross-communicate so as to sync indices over a larger group of caches. 
                - By having multiple hubs, rather than a single centralized index, the network is more resilient to downtime and censorship. Nevertheless this system increases the complexity of the network, and, relative to other possible solutions, still requires safeguards to prevent a malicious actor from using the network for denial-of-service attacks -- as an attacker only needs to attack a select few hubs rather than the entire network. This vulnerability decreases as more hubs are present in the network, and as more inter-hub communications are formed. 
             ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Fmyel%2Fz3MNoZWMdk.png?alt=media&token=8d18406c-6ddd-4c58-9374-e90f4329f345)
            - > Figure 4: Hubs maintain an index of cache-content pairs. In green we show communications where a cache informs a hub of the content they are hosting. In blue we show communications where a hub returns the cache address that holds the content a client is looking for. The client can then query that cache for the content. Hubs can also communicate to sync their indices such that each hub covers a larger group of caches. 
    - # Payments
        - In the Myel CDN implementation, payments are executed directly between cache providers and clients retrieving the content. To enable direct P2P payments, Myel uses existing crypto payment protocols.
        - Currently, the data transfer protocol is based on the Filecoin retrieval protocol and designed to operate in a trustless environment in which a provider may attempt to send bad content to a client and still receive payments. To that end, payments are divided into multiple intervals to let the client validate that the correct content is being transferred. If one of the parties decides to interrupt the transfer, the provider will still receive payments for some of the transferred blocks. As more blocks are successfully validated by the client, the payment interval can increase to include more blocks per payment and increase the speed of transfer.
         ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Fmyel%2F4ZFxA3bMjQ.png?alt=media&token=05b5fd9e-b95b-43bb-88cd-e36962fce2e1)
        - > Figure 3: Data transfers are batched into n data chunks. After the transfer of each chunk of data, the cache receives a fraction 1/n of the total payment. 
        - Centralized CDNs can currently handle up to 800 billion requests per day [8]. Each request on the Myel network generates n payment transaction messages. At the scale of centralized CDNs the network would need to handle  n x 800 billion messages a day. To avoid bottlenecks and make the system inherently scalable, most of those messages are generated off chain and accounted for in state channels that can be resolved on chain __asynchronously__. Those payment channels are created on the Filecoin blockchain, though other blockchains could be used to issue payments.
        - As mentioned previously, when content is transferred, payment happens reccuringly at an agreed upon payment interval. At this interval providers can pause transfers to request payment from the client, with the steps as follows: 
            - 1. The client signs and sends a voucher message with the requested amount. 
            - 2. The provider then stores those messages and can aggregate all the payments into a single blockchain transaction to redeem the payment. 
            - 3. Filecoin provides built in smart contracts to process payment vouchers and adds a 12h settlement period during which any of the two parties involved can submit valid vouchers. Once the settlement period is over, the provider can collect the resulting amount on their address.
        - __TODO: write about any lottery or lending system to improve payout experience__
- # **Economics**
    - Traditional centralized CDNs offer different pricing plans for organisations to pay for caching their content. Companies like Cloudflare charge fixed monthly plans per origin, that range from a free tier for hobby accounts on the global network to $20 (Pro), or $200 (Business) or custom pricing for enterprise. Advanced services such as Smart Routing are add-ons that can increase the monthly plans based on usage [10]. Others like Fastly charge a fixed fee per number of HTTP requests (i.e. $0.0075 per 10,000 requests) and another bandwidth fee per GB of content (i.e. $0.12 per 10TB of content served) on top of a base $50 monthly fee. Pricing also varies per region [11].
    - Unlike these systems, Myel clients (not publishers) pay a flexible retrieval fee per byte of content retrieved. Pricing can vary based on the region and based on demand at a given time. Further, similar to automated market maker protocols or applications like Uber, the protocol automates the price settings instead of providers manually setting their asks. This improves the user experience and guarantees market efficiency and incentives for providers to join the network. If a large amount of  providers request it, changes could be made to enable manual pricing.
    - Furthermore, like on the Cloudflare global network, some providers can join a higher latency network allowing retrievals at no cost for clients while maintaining priority for paid transfers. This permits new users to get familiar with the network before they start paying for transfers.
    - To begin with, hub nodes and caches will be rewarded from a community reward pool (consisting of Filecoin or another token). This way caches are rewarded for both uptime and for content delivery to begin with and the Myel network can sustain a large cache capacity even when minimal content has been onboarded.  We plan to gradually taper off these community rewards as content is onboarded to the network and content delivery generates sufficient returns for cache providers.
    - The average US household consumes 344GB of content on a monthly basis  [12]. At a cost of $0.005/GB, the average household would spend roughly $2/mo to retrieve content solely on the Myel network. If we assume that each cache provider services at least 20 users for **all of their content delivery needs**, they in turn would earn close to $40 monthly. 
    - > These consumption numbers are not reflective of traffic or pricing on the Myel network and we will update these numbers as we gather more data on consumption habits and optimal pricing levels.
- # **Case Study: Filecoin**
    - The Myel network is optimal for retrieving data from decentralized storage protocols [6, 7] such as Filecoin [6], in which content retrieval from a storage miner can be very slow. 
    - In the Filecoin decentralized storage protocol, clients select one or more nodes to use as storage providers and enter in codified trustless agreements to store a given piece of content. In this system, a storage miner can run very large infrastructure capable of storing data for thousands of different users. To be cost effective, storage miners tend to be located in areas with cheap real estate and cheap electricity which can be far from clients of the content. 
    - Further, if a high number of clients wish to retrieve content at the same time, even the most sophisticated infrastructure will face some bottlenecks and content delivery may slow down as a result. As such, this concentration of storage capabilities in non-densely populated areas can make retrieving content on the network very slow. 
    - The Myel network offers a way to address these issues. Clients run Myel nodes to access content stored by Filecoin miners. In this process they also become points of presence and get paid to cache content for other clients. This creates a secondary market for content retrievals in which stored content is cached close to end users and is delivered in exchange for micro-payments.
- # **Conclusion**
    - We've detailed the many parts of the Myel network and how they can be used to cache content in decentralized protocols, in particular Filecoin. These components are likely to change as the rollout of the network progresses. In the spirit of flexibility we are striving to make our development as modular as possible so that many designs for content delivery, content discovery, payments, and economics can be swapped out and tested rapidly.  
- # **References**
    - [1] Dykes, S. G., Jeffery, C. L., & Das, S. (1999). Taxonomy and design analysis for distributed Web caching. __Proceedings of the Hawaii International Conference on System Sciences__. 
    - [2] Almashaqbeh, G. (2019). __CacheCash: A Cryptocurrency-based Decentralized Content Delivery Network__. 
    - [3] Almashaqbeh, G., Kelley, K., Bishop, A., Cappos, J. (2019). CAPnet: a defense against cache accounting attacks on content distribution networks. __IEEE Conference on Communications and Network Security (CNS)__.
    - [4] https://rbtcollins.wordpress.com/2019/12/21/a-cachecash-retrospective/
    - [5] http://shareaza.sourceforge.net/mediawiki/index.php/Gnutella2
    - [6] Filecoin: A Cryptocurrency Operated File Storage Network, 2017.
    - [7] Arweave:  A Protocol for Economically Sustainable Information Permanence. 2018.
    - [8] Buterin, A next-generation smart contract and decentralized application platform (2014).
    - [9] https://www.fastly.com/press/press-releases/fastly-achieves-100-tbps-edge-capacity-milestone
    - [10] https://www.cloudflare.com/plans/
    - [11] https://www.fastly.com/pricing
    - [12] https://decisiondata.org/news/report-the-average-households-internet-data-usage-has-jumped-38x-in-10-years/
