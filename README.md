# MLCache
 Cover Letter
Dear chairs and anonymous reviewers, 

First of all, we would like to thanks to chairs for arranging the paper review. We also appreciate the constructive review comments from anonymous reviewers. 

We have studied the reviews carefully and revised the manuscript accordingly. The revisions to the comments are listed in details as follows. They are incorporated in the updated submission in blue fonts.

Sincerely,
Anonymous authors
=================================================================
Review 1
Comment 1: In abstract, the authors write” MLCache basically approximates the ideal model, and MLCache also ensures fairness and s SSD I/O latency.” What does “s SSD I/O latency” mean?
Response:
Thanks for the questions.  I have already made corrections in the submitted paper.

Comment 2: In the introduction part, the authors argue “NVMe SSDs provide the high-speed peripheral component interconnect express (PCIe) bus to enable high throughput (e.g., 126.03 GB/s).” Please provide the reference for this speed.   
Response:
Thanks. We have added relevant references. It is worth mentioning that the throughput of the PCIE5.0 standard formulated in 2019 has reached 128GB/s, and the throughput of the PCIE6.0 standard formulated in 2021 has reached 256GB/s.
 
Figure 1. I/O bandwidth of NVMe [1]
[1] Announces upcoming pcie xpress ®6.0 specification. https://mms.businesswire.com/media/20190618005945/en/727854/5/Option3.jpg?download=1

Comment 3: The authors argue that “NVMe protocol becomes a promising solution to replace the conventional serial advanced technology attachment (SATA) protocol in modern SSDs”. Since SATA is an interface and NVMe is a protocol, is it possible to integrate the SATA interface with NVMe protocol? If no, please provide the reasons.
Response:
We should clarify several concepts. At the physical interface level, there are two physical interfaces, SATA and PCIe. 
SATA（Serial ATA）is the next -generation internal storage interconnect, designed to replace parallel ATA technology. Serial ATA is the proactive evolution of the ATA interface from a parallel bus to a serial bus architecture. SATA is the market incumbent and dominant interface for connecting an SSD to the PC. It employs the command protocol AHCI (it also supports IDE) which was built with slower spinning disks in mind rather than flash memory. The physical throughput limit of SATA is 6Gb/s
PCIe（PCI Express）is a high performance, general purpose I/O Interconnect defined for a wide variety of future computing and communication platforms. Key PCI attributes, such as its usage model, load -store architecture, and software interfaces, are maintained, whereas its parallel bus implementation is replaced by a highly scalable, fully serial interface. PCie6.0 already supports throughput up to 256Gb/s.
NVMe (Non-Volatile Memory Express) is an open logical device interface specification for accessing non-volatile storage media attached via a PCIe bus. NVMe is a fairly new protocol for accessing high-speed storage media that brings many advantages compared to legacy protocols. 
Unlike AHCI(SATA), the NVMe architecture did not need to consider legacy interfaces and associated software stacks with which backward compatibility had to be maintained. This has allowed an interface design that is optimized to take advantage of those very characteristics that make NAND flash-based devices so attractive. 
a) The interface allows multiple command submission/completion pathways, or IO channels, based on submission/completion queue pairs. An IO channel is composed of one or more submission queues associated with a particular completion queue. 
b) Multiple submission and completion queues can be created dynamically. Up to 64K submission and completion queues can be supported, each up to 64K entries deep. 
c) Priorities can be assigned to queues. NVMe provides support for “simple round robin”, “weighted round robin with urgent priority” or a vendor-defined priority scheme.
In summary, the NVMe protocol is developed around the PCIe interface. So NVMe cannot be applied to SATA interface.

Comment 4: This paper presents a neural network in the SSD controller, which usually requires high computational resources. Usually, the SSD controller has limited computational capability. So, is it possible to integrate this neural network in the SSD controller?
You argue “The computational cost is 110592 floating-point operations; the time required is 0.220743ms”. How do you get these data? Do you really run your network in the SSD controller?
Response:
Neural network is generally considered to be expensive mainly because of its training process, which involves gradient descent (GD) and back propagation (BP), which results in a very large amount of calculation when training neural networks. But the calculation process that does not include training only uses very simple matrix operations and function mapping. Our very small overhead is mainly due to the following two key factors:
1. The neural network we use is pre-trained, and MLCache does not require online training during running, which eliminates a lot of overhead.
2. What we use is a simple shallow neural network, not a complex deep neural network. The number of MLCache neurons and neural network layers is very small, the number of layers is at most 2, and the number of neurons does not exceed 256.
In addition to low overhead, we also know that the processor speed of SSD is fast enough. As we mentioned in the paper, “Most ﬂash memory masters base on NVMe1.3 and above use Cortex R5 based chips for design. Take the consumer-level master Phison E12 produced in the fourth quarter of 2018 as an example. It has 2 Cortex R5 cores, and its average operating speed is above 300MHZ.” This processor speed is sufficient to handle neural network operations. 
Let's calculate the cost of the largest neural network. Our neural network for 16 streams is as follows. Our largest neural network input is 160, contains two hidden layers, each contains 256 hidden layers of neurons, and the output unit is a neural network with 16. According to the principle of neural network, it can be known that its computational cost is 160*256+256*256+256*16=110592 floating-point operations.
The neural network of each layer calculates the floating number as num_inputs x num_outputs. This is the calculation principle of neural network, because the trained neural network is just a simple weight calculation and addition, the FLOPs of the neural network can be easily evaluated, and the estimated computational cost will hardly deviate from the actual one. 
The real CPU time is very difficult to obtain and measure, so we derive the time according to the CPU calculation model. “0.220743mss” refers to a conservatively estimated time, which is calculated by us through CPU frequency and FLOPs. Our purpose of proposing this time is to illustrate the following sentence, "which is controlled within 1ms". Because most of the calculation process of the neural network is pure floating-point operations, the estimated time is relatively accurate. Another thing to note is that 0.220743ms is just a very conservative estimate for us. This is when only a single core is used and the selected operating rate is the lowest. Actually, in the case of multi-core and higher operating CPU frequencies, the calculation time will only be less. In order to avoid misunderstandings for readers, we have revised this sentence in the original text.

Comment 5: In Section III.B MLCache, you should delete the following contents: “(!!!!we should simply introduce the ideas behind PARDA in background!!!!)”
Response:
Thank you for your reminder. It has been modified in the paper.

Comment 6: The concept of stream is not very clear to me? Do you mean the requests belong to the same queue? Why do we need many queues? If we only use one request queue, what will happen?
Response:
Let's first understand the definition of NVMe for streaming.
The Streams Directive enables the host to indicate (i.e., by using the stream identifier) to the controller that the specified logical blocks in a write command are part of one group of associated data. This information may be used by the controller to store related data in associated locations or for other performance enhancements. The concept of stream is shown in Figure 1.
 
Figure 1 The concept of stream [1].
All data associated within a stream is expected to be invalidated at the same time (e.g., updated, trimmed, unmapped). And we can align NAND block allocation based on application data characteristics (e.g., update frequency). Since data separation decisions can be made more intelligently on the host level over on the SSD level, when streams are properly managed, they can signiﬁcantly improve both the performance and lifetime of ﬂash-based SSDs. With a multi-stream SSD, FTL can store data of diﬀerent temperatures to diﬀerent streams so that data in the same erase blocks can be invalidated closely together and achieve low GC overhead. So, stream provides better endurance, improved performance, and consistent latency.
Queue and stream are two completely different concepts. Stream is designed to enhance the relevance of data. For example, common methods include setting different tenant requests to different streams, or setting different application requests to different streams. The queue aims to enhance the concurrency of SSD. The concept of queue is shown in Figure 2.
 
Figure 2 The position of queue in NVMe [2].
If there is only one stream, this stream will monopolize all cache resources and other SSD resources.
[1] Yang, Jingpei, et al. "AutoStream: automatic stream management for multi-streamed SSDs." Proceedings of the 10th ACM International Systems and Storage Conference. 2017.
[2] NVMe 1.4a speciﬁcation. https://nvmexpress.org/developers/nvmespeciﬁcation/, 2020.


Comment 7: A good neural network needs to be well trained. Does your datasets cover a wide range of real data workloads?
Response:
We have added this part of the content in Section Ⅲ.B
Training data: We use randomly synthesized traces as our main training data, accounting for nearly 95%. We can add random locality requests to our synthetic traces. In the synthetic trace, we can add any combination of locality, we can arrange and combine the combination of locality as much as possible, so as to ensure sufficient diversity of training data. We use Microsoft Research Cambridge traces as our remaining 10% of training data. Microsoft Research Cambridge traces is tested by researchers as traces because of its wide range of data features and high credibility. Therefore, our test data is sufficient and complete. Randomly generating traces can ensure that the data characteristics of our different streams are different. We have conducted additional tests. Even if we use randomly generated traces as training data, we can still achieve the same level of test results.
The reason for using a large amount of synthetic data is that one is because real data workloads are limited, and the other is that random synthetic data workloads can actually include real data workloads. Because we let the random synthetic data workloads contain the arrangement of local features as much as possible.
Test Data:
1. The test-time data set we use includes real data sets and synthetic data sets. Table 1 in the paper has shown our test data. The synthetic data comes from the trace generation function of MQSim.
2. It is worth mentioning that the selection of trace is randomly selected from Microsoft Research Cambridge traces. In addition, our test data and training data are completely non-overlapping, which avoids the influence of training data on the test data, which is also an important criterion for neural network testing.

Comment 8: Some of the data has timing series connection. If you use RNN-based neural network, can you achieve better results?
Response:
Thanks for your comment. Indeed, with respect to the cache division of the data has timing series connection, it is more appropriate to use RNN. Our plan currently only considers general workloads. We will add the cache partition problem of the data has timing series connection to our future work. Sincerely thank you for your suggestions.

Comment 9: In your experiment, how many queues do you use and how do you configure the cache?
Response:
In the experiment, the number of queues is set to 8, and the number of pages configured in the cache is 220, 240, and 680 in the 2, 4, and 8 streams, respectively. 
In the paper we mentioned that we set such a small cache page mainly to simulate scenarios where different cache resources are scarce.
Some reviewers mentioned that it is recommended to use a 256MB cache for experimentation. We also did a corresponding supplementary experiment. The experiment still proves that MLCache can effectively approximate the optimal allocation scheme. If you are interested, you can take a look at our supplementary experiment.

Review 2
Comment 1: The authors mentioned that the inputs of the machine learning module within the MLCache are data reuse distance and frequency. However, in the experimental section, the authors only analyzed the reuse distance of Trace 16. The reviewer wonders could the author provide more results of reuse distance by analyzing different workloads. It can prove that the reuse distance of most workloads can be cached by DC cache.
Response:
As you can see, we use Trace 16 as an example in the paper, and use Figure 11 for analysis. Figure 11 a) actually contains not only the information about the reuse distance, but also the frequency information. In Figure 11 a), each line represents the relative number of different reuse distance ranges, that is, the same color line and the height of the line contain the reuse distance information and frequency information, respectively. Because we observe the reuse distance in the last n requests, the number of maximum reuse distance ranges for each stream adds up to n, so each curve contains frequency information. For example, in the period ③ in Fig. 11 a), the relative frequency of each reuse distance range of proj_0 is higher than src2_2.
As for why we only show Trace 16 because the paper has a limit on the number of pages. We can provide you with more sample analysis of trace.
  
 

Comment 2: According to Page 6, the author said that the loss is the squared difference between the predicted value and the true label value, and the label value is obtained by the real optimal distribution solution. To the reviewer’s knowledge, the label value varies when the workloads are different. Thus, could the authors explain how to get the label value at runtime for constructing the machine learning model?
Comment 3: Additionally, could the authors explain the real optimal distribution solution in detail?
Response:
Thanks for your comment. The second and third questions are essentially one question. We will answer how to obtain the optimal algorithm and how to train the neural network.
We have added an explanation of the optimal allocation solution in Section IV.A
First of all, our optimal allocation scheme is obtained when all the complete traces are known. Just like the introduction to the reuse distance we started first, after having the complete reuse distance information of the trace, we can accurately calculate whether each request arrives at a certain capacity or not. By calculating the complete reuse distance, we can accurately calculate the optimal allocation plan. However, please note that there are two points to note here:
1. The optimal allocation scheme can see all future traces to achieve the optimal allocation plan.
2. We will calculate the number of hits for each stream under different capacities, so that we can get the partition scheme with the highest number of hits through combination attempts. Another point is that calculating the optimal allocation plan in this way is an np complete problem, requiring the processor to consume a lot of time for calculations.
As for the second question you mentioned, how do we get the tag value after changing the workload. What we want to explain is:
1. The model we trained is the relationship between reuse distance and frequency and the best allocation plan. We have explained in the background that reuse distances and relative frequencies are directly related to the best allocation plan. We trained on a generic model that was workload independent, so changing the workload did not require us to retrain.
2. The MLCache module we use in SSD has been trained. We have obtained a good neural network model between the reuse distance and frequency and the best allocation scheme in the training phase. During the run, there is no training again, so we don't need to get the label during running.

Comment 4: On Page 6, the authors claimed no overfitting problem in the MLCache’s machine learning model because there are no abnormal samples. However, the overfitting problem in machine learning is not only caused by abnormal samples. There are some different factors (e.g., model complexity and the number of training data) that cause an overfitting problem. Hence, how do the authors know that there is not overfitting problem in their model?
Response:
There are indeed many other factors that cause over-fitting. We apologize for not explaining in detail in the paper. We have revised the corresponding description in the original text. The three most important factors of overfitting are model complexity, too little training data, and noise data.
1. Model complexity. Complex neural networks are prone to overfitting. Therefore, we only need to find the simplest neural network that can meet the expected requirements. At first, we adopted the simplest single hidden layer, which contained a neural network with a small number of neurons, and then continued to increase the depth and width of the neural network until the test results could meet our expectations. The neural network obtained in this way can avoid overfitting caused by complex models as much as possible.
2. Overtraining. When the training data is small, it is likely to overtrain to make the neural network fit the training data. But when the training data is enough, we won't have the problem of overtraining. MLCache uses a large number of synthetic traces as training data, and we have enough training data to ensure that there will be no overtraining. From the results, the neural network trained by synthetic trace has a good effect in real trace testing.
3. Noise data. We have explained in the paper that we can calculate the optimal solution and avoid the noise data in the training data.
The above clarified the basic elements of over-fitting, the following analysis from the results of why there is no over-fitting in MLCache.
Although we have excluded the above basic elements, we still have to admit that in rare cases, some examples have over-fitting. We will continue to pay attention to overfitting. Thanks again for your reminder.

Comment 5: All data from writing requests will be cached in the DC cache. How do the authors guarantee the data consistency and reliability while power loss occurs?
Response:
We clarified it in the revision (Section Ⅱ.F ). 
DRAM is widely used in SSD as a cache to improve the overall performance of SSD, but volatile storage has data loss in the event of a power failure. There are several ways for SSD to prevent the loss of important data in the device cache:
a) Traditional battery-backed DRAMs[1,2,3].
b)) A cap that provides enough power to flush all of the dirty data in the device cache to NAND flash memory[4,5,6].
c) Use high-performance non-volatile storage to replace DRAM, such as ferroelectric RAM (FRAM) and phase change RAM (PRAM) [7,8].
[1]H. Shim, B. Seo, J. Kim and S. Maeng, "An adaptive partitioning scheme for DRAM-based cache in Solid State Drives," 2010 IEEE 26th Symposium on Mass Storage Systems and Technologies (MSST), 2010, pp. 1-12, doi: 10.1109/MSST.2010.5496995.
[2] Kateja, Rajat, et al. "Viyojit: Decoupling battery and DRAM capacities for battery-backed DRAM." ACM SIGARCH Computer Architecture News 45.2 (2017): 613-626.
[3] Moshayedi, Mark, and Patrick Wilkison. "Enterprise SSDs: Solid-state drives are finally ready for the enterprise. But beware, not all SSDs are created alike." Queue 6.4 (2008): 32-39.
[4] Lee, Hyeon Gyu, et al. "SpartanSSD: a reliable SSD under capacitance constraints." 2021 IEEE/ACM International Symposium on Low Power Electronics and Design (ISLPED). IEEE, 2021.
[5] Kim, Dongwook, and Sooyong Kang. "Dual region write buffering: Making large-scale nonvolatile buffer using small capacitor in SSD." Proceedings of the 30th Annual ACM Symposium on Applied Computing. 2015.
[6]How micron SSDS handle unexpected powerloss. https://www.micron.com/-/media/client/global/documents/products/white-paper/ssd_power_loss_protection_white_paper_lo.pdf
[7] Farbeh, Hamed, and Nezam Rohbani. "PCM-oriented cache management strategies for solid-state disks." 2018 Real-Time and Embedded Systems and Technologies (RTEST). IEEE, 2018.
[8] Tarihi, Mojtaba, et al. "A hybrid non-volatile cache design for solid-state drives using comprehensive I/O characterization." IEEE Transactions on Computers 65.6 (2015): 1678-1691.

Comment 6: The workloads used in the experiments were released in 2008, thirteen years ago. Currently, more block traces have been released and uploaded to the SINA IOTTA website. Could the authors utilize more recent workloads to evaluate the capabilities of the proposed solution?
Response:
Thanks for the comment.  TraceTracker updated the data set of Microsoft Research Cambridge in SINA IOTTA in 2017[1]. At the same time, a large number of scientific researches in recent years have also adopted these data sets. In Fast’2021 alone, there are at least three papers using Microsoft Research Cambridge's trace [2,3,4]. In addition, there are a large number of recent papers using this data set [5,6].
[1] Kwon, Miryeong, et al. "TraceTracker: Hardware/software co-evaluation for large-scale I/O workload reconstruction." 2017 IEEE International Symposium on Workload Characterization (IISWC). IEEE, 2017.
[2] Rodriguez, Liana V., et al. "Learning Cache Replacement with {CACHEUS}." 19th {USENIX} Conference on File and Storage Technologies ({FAST} 21). 2021.
[3] Liu, Zhang, et al. "eMRC: Efficient Miss Ratio Approximation for Multi-Tier Caching." 19th {USENIX} Conference on File and Storage Technologies ({FAST} 21). 2021.
[4] Jiang, Tianyang, et al. "FusionRAID: Achieving Consistent Low Latency for Commodity {SSD} Arrays." 19th {USENIX} Conference on File and Storage Technologies ({FAST} 21). 2021.	
[5] Wang, Yuchen, Junyao Yang, and Zhenlin Wang. "Dynamically Configuring LRU Replacement Policy in Redis." The International Symposium on Memory Systems. 2020.
[6] Liu, R., Liu, D., Chen, X., Tan, Y., Zhang, R., & Liang, L. (2021). Self-Adapting Channel Allocation for Multiple Tenants Sharing SSD Devices. IEEE Transactions on Computer-Aided Design of Integrated Circuits and Systems.

Comment 7: In the experimental settings, the authors set the cache capacity to 260 pages in 4 streams and 680 pages in 8 streams. However, the authors do not say why. Could the authors include some real product specifications for the experimental settings?
Response:
Our original idea of choosing this cache size was due to the following reasons.
1. DRAM is not only used as cache in SSD, but SSD is also used extensively in ECC and GC operations [1,2,3]. In addition, there are a lot of metadata stored in DRAM, such as flash page state table (FPST) and flash block state table (FBST) and so on [7]. Many calculations at the FTL level also use DRAM [4,5,6]. Although the manufacturer clearly stated the capacity of the DRAM, it did not clearly specify the size of the cache. Therefore, the size of the cache is set by ourselves. Below we will also explain that the cache capacity will not affect the performance of the algorithm.
2. Cache capacity does not affect MLCache. The size of the capacity only affects the maximum value of the reuse distance. While MLCache inputs all the normalized reuse distances into the neural network, the neural network in MLCache focuses on the reuse distance distribution, not the size of the reuse distance. In order to dispel the doubts of the reviewers, we added a comparative experiment under 256MB. The experimental results prove that MLCache can still effectively optimize the cache partition.
3.When the size of the cache space is very large, the cache partition algorithm becomes meaningless, which is actually related to the locality of traces. When the cache space is scarce, it can better reflect the pros and cons of the cache partitioning algorithms. When the cache capacity is too large, the cache is sufficient to meet the needs of all competitors, and the results obtained by the optimal cache allocation scheme and the benchmark scheme will also be similar, and it is difficult to observe the advantages of the cache partition algorithm. In addition, various manufacturers have not actually announced the DRAM capacity used for cache, so we set a smaller capacity cache for comparison experiments.
In order to dispel the reader's doubts, we also added the experiment under the 256MB cache capacity.

We also added capacity descriptions and supplementary experimental descriptions to the paper.
[1] Yan, Z., Jiang, H., Jiang, S. and Tan, Y., 2019, May. SES-Dedup: Scrambler-Resistant ECC-based SSD Deduplication. In Proceedings of the 35th International Conference on Massive Storage Systems and Technology (MSST'19).
[2] Y. Qin, D. Feng, J. Liu, W. Tong, Y. Hu and Z. Zhu, "A Parity Scheme to Enhance Reliability for SSDs," 2012 IEEE Seventh International Conference on Networking, Architecture, and Storage, 2012, pp. 293-297, doi: 10.1109/NAS.2012.40.
[3] Debnath, B., Krishnan, S., Xiao, W., Lilja, D.J. and Du, D.H., 2011, May. Sampling-based garbage collection metadata management scheme for flash-based storage. In 2011 IEEE 27th Symposium on Mass Storage Systems and Technologies (MSST) (pp. 1-6). IEEE.
[4] Fareed, I. et al. "Update Frequency-directed Sub-page Management for Mitigating Garbage Collection and DRAM Overheads." IEEE Transactions on Computer-Aided Design of Integrated Circuits and Systems PP.99(2021):1-1.
[5] Zhang, J., Shihab, M. and Jung, M., 2014. Power, energy, and thermal considerations in ssd-based i/o acceleration. In 6th {USENIX} Workshop on Hot Topics in Storage and File Systems (HotStorage 14).
[6] Chen, F., Luo, T. and Zhang, X., 2011, February. CAFTL: A Content-Aware Flash Translation Layer Enhancing the Lifespan of Flash Memory based Solid State Drives. In FAST (Vol. 11, pp. 77-90).
[7] Kgil, T., Roberts, D. and Mudge, T., 2008, June. Improving NAND flash based disk caches. In 2008 International Symposium on Computer Architecture (pp. 327-338).

Comment 8: On page 13, the authors mentioned one solution called CloudCache that uses workload reuse intensity to estimate cache space requirements. Could the authors compare the proposed solution with this solution?
Response:
The goal of CloudCache is to use optimized cache partitions to reduce cache capacity. CloudCache sacrifices cache hits to reduce the size of the cache, so the hit rate of CloudCache will be lower than the baseline, because CloudCache uses SSD as the cache, and its more prominent is the reduction of cache capacity. We use DRAM as the cache, so we don't need to consider the reason of its lifetime, so our goal is to maximize the hit rate. Its optimization goal is inconsistent with ours, and it is not a good comparison algorithm. Because CloudCache's hit rate is lower than the baseline, our effect must be better than CloudCache.

Comment 9: Minor problems:
On page 2, the sentence in the summary of contribution is repetitive.
On page 5, some notes have not been removed from the draft. 
The authors should carefully proofread the manuscript.
Response:
Thanks for your reminder, we have corrected it in the corresponding position

Review 3
Comment 1: Caching write data in DRAM in an SSD creates a fundamental problem for SSDs. Since SSDs are storage devices, all data has to be written to persistent medium. Otherwise, the data can be losing if there is a power failure. Therefore, caching write data in DRAM to reduce the amount of data written to flash memory cannot be an acceptable approach.
Response:
We clarified it in the revision (Section Ⅱ.F ). 
DRAM is widely used in SSD as a cache to improve the overall performance of SSD, but volatile storage has data loss in the event of a power failure. There are several ways for SSD to prevent the loss of important data in the device cache:
a) Traditional battery-backed DRAMs[1,2,3].
b)) A cap that provides enough power to flush all of the dirty data in the device cache to NAND flash memory[4,5,6].
c) Use high-performance non-volatile storage to replace DRAM, such as ferroelectric RAM (FRAM) and phase change RAM (PRAM) [7,8].
DRAM is widely used as a cache in SSDs, and DRAM can effectively improve the overall performance of SSDs. Enterprise SSDs generally use backup power or capacitors to prevent the loss of DRAM data. In the white paper “How micron SSDs handle unexpected power failed”, micron clearly explained how micron SSDs uses capacitors to prevent DRAM data loss caused by unexpected power failure [6]. The chapter we added also explains in detail how companies can prevent accidental power outages.
Your question later also mentioned DRAMless ssd. In fact, DRAMless ssd may still face the risk of data loss after power failure. Because many DRAMless SSDs use host layer DRAM for data caching in order to avoid performance problems. DRAMless are being widely spread for cheap client SSD and embedded SSD markets in recent years because they are cheap and consume less power. Obviously, their performance is lower than conventional SSDs because they cannot exploit advantages of DRAM in the controller. However, this problem can be alleviated by using host memory buffer (HMB) feature of Non-Volatile Memory Express (NVMe), which allows SSDs to utilize the DRAM of host. In this paper, we show that commercial DRAM-less SSDs clearly exhibit worse I/O performance than SSDs with internal DRAM, but this can be improved by using the HMB feature. 
[1]H. Shim, B. Seo, J. Kim and S. Maeng, "An adaptive partitioning scheme for DRAM-based cache in Solid State Drives," 2010 IEEE 26th Symposium on Mass Storage Systems and Technologies (MSST), 2010, pp. 1-12, doi: 10.1109/MSST.2010.5496995.
[2] Kateja, Rajat, et al. "Viyojit: Decoupling battery and DRAM capacities for battery-backed DRAM." ACM SIGARCH Computer Architecture News 45.2 (2017): 613-626.
[3] Moshayedi, Mark, and Patrick Wilkison. "Enterprise SSDs: Solid-state drives are finally ready for the enterprise. But beware, not all SSDs are created alike." Queue 6.4 (2008): 32-39.
[4] Lee, Hyeon Gyu, et al. "SpartanSSD: a reliable SSD under capacitance constraints." 2021 IEEE/ACM International Symposium on Low Power Electronics and Design (ISLPED). IEEE, 2021.
[5] Kim, Dongwook, and Sooyong Kang. "Dual region write buffering: Making large-scale nonvolatile buffer using small capacitor in SSD." Proceedings of the 30th Annual ACM Symposium on Applied Computing. 2015.
[6]How v handle unexpected powerloss. https://www.micron.com/-/media/client/global/documents/products/white-paper/ssd_power_loss_protection_white_paper_lo.pdf
[7] Farbeh, Hamed, and Nezam Rohbani. "PCM-oriented cache management strategies for solid-state disks." 2018 Real-Time and Embedded Systems and Technologies (RTEST). IEEE, 2018.
[8] Tarihi, Mojtaba, et al. "A hybrid non-volatile cache design for solid-state drives using comprehensive I/O characterization." IEEE Transactions on Computers 65.6 (2015): 1678-1691.

Comment 2: The paper does not indicate which address mapping scheme they are considered. They have indicated page-level, block-level, and a hybrid mapping for address mapping. However, in the rest of the paper, they did not mention the type of address mapping they are considering. This is critical since each type of address mapping has it unique issue to be investigated.
Response:
What we need to reiterate is that DFTL is page-level mapping, and DFTL has been widely used in current flash memory, and DFTL is the most representative mapping algorithm [1,2]. The most advanced SSD simulator MQSim uses DFTL based on page mapping [3]. We introduced DFTL in detail in Section II.C and explained that it is actually based on page mapping. We will add DFTL and page-level mapping instructions in the introduction.
For block-level mapping, it occupies a very small space and can completely store data in DRAM, so there is no need for cache partitioning. As for the hybrid mapping, its implementation methods are diverse, and some do not need to use DRAM for caching, which are closely related to the specific implementation methods.
In addition, MLCache is a universally available cache partitioning algorithm. MLCache actually only focuses on the reuse distance distribution and relative frequency. As long as the cache uses the LRU algorithm, MLCache can run well, so page-level, block-level, and a hybrid mapping as long as the LRU algorithm is used to control the elimination in the cache, we can use MLCache for optimization.
Therefore, MLCache is optimized based on the most widely used DFTL mapping scheme.
[1] ZHANG, Peiyong; TANG, Huanjie. High‐efficient superblock flash translation layer for NAND flash controller. Electronics Letters, 2020, 56.6: 278-280.
[2] Y. Pan, Y. Li, H. Zhang, H. Chen and M. Lin, "GFTL: Group-Level Mapping in Flash Translation Layer to Provide Efficient Address Translation for NAND Flash-Based SSDs," in IEEE Transactions on Consumer Electronics, vol. 66, no. 3, pp. 242-250, Aug. 2020, doi: 10.1109/TCE.2020.2991213.
[3] Tavakkol, Arash, et al. "Mqsim: A framework for enabling realistic studies of modern multi-queue {SSD} devices." 16th {USENIX} Conference on File and Storage Technologies ({FAST} 18). 2018. 

Comment 3: The authors do present a design of neural network. However, it is not clear how this neural network is operating and how the desired output (% of space for each stream) can be accomplished. It is also not clear on the justification of using a neural network, especially, the neural network will be trained only periodically.
Response:
The related process of neural network has been added to Section Ⅲ. B on page 7.
As for the rationality of using neural networks, as mentioned in your fourth question, the optimal cache partitioning algorithm is actually very complicated to design. This is indeed the case. The optimal cache partition is actually an NP-complete problem, and various current optimization algorithms try to optimize this NP-complete problem. We know that neural networks are very suitable for solving complex model problems, and neural networks are also one of the important solutions to NP-complete problems. So, when we first faced this problem, our first thought was, why not try to use neural networks. So, we are the first researcher to propose the use of neural networks to solve the optimal partition problem. The experimental results also show that the neural network is indeed suitable for solving such problems.
MLCache does not periodically train neural networks. Here are two related facts about MLCache:
1. MLCache has trained the neural network in advance, and MLCache will not train the neural network when the SSD is running.
2. MLCache periodically allocates the cache, which can avoid short-term request fluctuations causing a large number of cache evictions. Any cache partition algorithm will allocate cache periodically, instead of changing the cache partition every moment.
The model we trained is the relationship between reuse distance and frequency and the best allocation plan. We have explained in the background that reuse distances and relative frequencies are directly related to the best allocation plan. We trained on a generic model that was workload independent, so changing the workload did not require us to retrain.

Comment 4: What is the optimal algorithm that the authors referred to in their experiments? Not enough discussion was presented in the paper. Is this really an optimal algorithm? My impression is that such an algorithm can be extremely hard to design.
Response:
We have added an explanation of the optimal allocation solution in Section IV.A
First of all, our optimal allocation scheme is obtained when all the complete traces are known. Just like the introduction to the reuse distance we started first, after having the complete reuse distance information of the trace, we can accurately calculate whether each request arrives at a certain capacity or not. By calculating the complete reuse distance, we can accurately calculate the optimal allocation plan. However, please note that there are two points to note here:
1. The optimal allocation scheme can see all future traces to achieve the optimal allocation plan.
2. We will calculate the number of hits for each stream under different capacities, so that we can get the partition scheme with the highest number of hits through combination attempts. Another point is that calculating the optimal allocation plan in this way is an np complete problem, requiring the processor to consume a lot of time for calculations.

Comment 5: The paper did not explicitly mention the DRAM size that they are used in their experiments. I can find the following text “We set the cache capacity to 260 pages in the 4-streams and 680 pages in the 8-streams. We set the number of CMT cache items to 10 times the number of data cache items.” Assuming 8KB/page, the total DC size is about 2 MB for the case of 4-streams. Since the number of CMT cache items (I would assume each item is the mapping information of one page) is 10 times the number of data cache items. Assuming each mapping information of a page is 8 Bytes (four bytes for LPA and another 4 Bytes for PPA), each page of 8KB can hold 1000 items. Therefore, 2,600 items only need 3 pages (i.e., 24 KB) to hold. I am not sure what is the impact of caching results with this smaller amount of DRAM. Among the existing SSDs, there is one type of cheap SSD without any DRAM is called DRAMless. More expensive and faster SSDs may have up to 256 MB of DRAM space.
Response:
Our original idea of choosing this cache size was due to the following reasons.
1. DRAM is not only used as cache in SSD, but SSD is also used extensively in ECC and GC operations [1,2,3]. In addition, there are a lot of metadata stored in DRAM, such as flash page state table (FPST) and flash block state table (FBST) and so on [7]. Many calculations at the FTL level also use DRAM [4,5,6]. Although the manufacturer clearly stated the capacity of the DRAM, it did not clearly specify the size of the cache. Therefore, the size of the cache is set by ourselves. Below we will also explain that the cache capacity will not affect the performance of the algorithm.
2. Cache capacity does not affect MLCache. The size of the capacity only affects the maximum value of the reuse distance. While MLCache inputs all the normalized reuse distances into the neural network, the neural network in MLCache focuses on the reuse distance distribution, not the size of the reuse distance. In order to dispel the doubts of the reviewers, we added a comparative experiment under 256MB. The experimental results prove that MLCache can still effectively optimize the cache partition.
3.When the size of the cache space is very large, the cache partition algorithm becomes meaningless, which is actually related to the locality of traces. When the cache space is scarce, it can better reflect the pros and cons of the cache partitioning algorithms. When the cache capacity is too large, the cache is sufficient to meet the needs of all competitors, and the results obtained by the optimal cache allocation scheme and the benchmark scheme will also be similar, and it is difficult to observe the advantages of the cache partition algorithm. In addition, various manufacturers have not actually announced the DRAM capacity used for cache, so we set a smaller capacity cache for comparison experiments.
In order to dispel the reader's doubts, we also added the experiment under the 256MB cache capacity.


We also added capacity descriptions and supplementary experimental descriptions to the paper.
As for the DRAMless SSD you mentioned , DRAMless SSD are being widely spread for cheap client SSD and embedded SSD markets in recent years because they are cheap and consume less power. Obviously, their performance is lower than conventional SSDs because they cannot exploit advantages of DRAM in the controller. However, this problem can be alleviated by using host memory buffer (HMB) feature of Non-Volatile Memory Express (NVMe), which allows SSDs to utilize the DRAM of host. DRAM-less SSDs clearly exhibit worse I/O performance than SSDs with internal DRAM [9]. 
[1]H. Shim, B. Seo, J. Kim and S. Maeng, "An adaptive partitioning scheme for DRAM-based cache in Solid State Drives," 2010 IEEE 26th Symposium on Mass Storage Systems and Technologies (MSST), 2010, pp. 1-12, doi: 10.1109/MSST.2010.5496995.
[2] Kateja, Rajat, et al. "Viyojit: Decoupling battery and DRAM capacities for battery-backed DRAM." ACM SIGARCH Computer Architecture News 45.2 (2017): 613-626.
[3] Moshayedi, Mark, and Patrick Wilkison. "Enterprise SSDs: Solid-state drives are finally ready for the enterprise. But beware, not all SSDs are created alike." Queue 6.4 (2008): 32-39.
[4] Lee, Hyeon Gyu, et al. "SpartanSSD: a reliable SSD under capacitance constraints." 2021 IEEE/ACM International Symposium on Low Power Electronics and Design (ISLPED). IEEE, 2021.
[5] Kim, Dongwook, and Sooyong Kang. "Dual region write buffering: Making large-scale nonvolatile buffer using small capacitor in SSD." Proceedings of the 30th Annual ACM Symposium on Applied Computing. 2015.
[6]How v handle unexpected powerloss. https://www.micron.com/-/media/client/global/documents/products/white-paper/ssd_power_loss_protection_white_paper_lo.pdf
[7] Farbeh, Hamed, and Nezam Rohbani. "PCM-oriented cache management strategies for solid-state disks." 2018 Real-Time and Embedded Systems and Technologies (RTEST). IEEE, 2018.
[8] Tarihi, Mojtaba, et al. "A hybrid non-volatile cache design for solid-state drives using comprehensive I/O characterization." IEEE Transactions on Computers 65.6 (2015): 1678-1691.
[9] Kim, K., & Kim, T. (2020). HMB in DRAM-less NVMe SSDs: Their usage and effects on performance. PloS one, 15(3), e0229645. 

Comment 6: The calculation of reuse distance was done by PARDA. However, I cannot find a definition of reuse distance in the paper. (The typical reuse distance between adjacent accesses to the same paper excluded the number of redundant accesses to the same pages. I am not sure what is used in this paper. I did find some comments on using PARDA as shown in “MLCache uses PARDA (!!!!we should simply introduce the ideas behind PARDA in background!!!)” However, no information about PARDA is provided.
Response:
We have added information about reuse distance and Parda in Section Ⅰ and Section Ⅱ.B

Comment 7: The baseline used in experimental section is referred to an even partition of DC space for each stream. Another obvious baseline should be no partition at all. That is, DC space will be used and shared by all streams.
Response:
Here is why we did not consider comparing shared cache algorithms.
1. From the beginning of the paper, we have proposed that we are actually stream-oriented optimization. Stream is a collection of requests with a similar life cycle. From another perspective, the life cycle is actually reuse distance. Similar life cycles have many benefits for SSDs. For example, for data allocated on a block, we use the reuse distance to cache the streams to actually ensure that each cache retains data with an approximate life cycle. When data is evicted, requests with similar life cycles will also be evicted, making the effect of stream more stable.
If multi-stream SSD adopts a shared cache solution, competition between multiple streams will cause the life cycle of write-back data to be no longer similar. This will cause a large amount of messy life cycle data to be stored in the block, which will eventually cause write amplification and shorten the lifespan.
2. As MQSim and we mentioned in IV.E, independent caching can effectively guarantee the fairness of the stream and the quality of service.
In summary, MLCache is essentially a partitioned cache algorithm for NVME streams. The sharing algorithm is completely uncontrollable for the fairness and service quality of the stream, and its application to the stream is unfavorable.
Shared and independent cache partition schemes are different choices in different scenarios. Enterprises can choose shared cache or independent partitioned cache according to their own needs. If an enterprise needs better fairness and high quality of service, or in a multi-tenant cloud business scenario, it can only choose an independent cache partition scheme.
Therefore, the fully shared cache is not a suitable comparison object for MLCache, and its comparison is beyond the scope of this paper.

Comment 8: The eviction policies for either DC or CMT are not stated in the paper. I would guess they both are related to LRU. It will be better if the authors can clearly state them.
Response:
We have added a description in Section Ⅰ.
