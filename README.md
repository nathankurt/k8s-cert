# k8s-cert

## Notes
For Notes, start [In the Notes Folder!](/notes) and [Core Concepts](/notes/core-concepts.md) for in depth notes. 

* [core-concepts.md](/notes/core-concepts.md) Starts with the [Core-Concepts](/notes/core-concepts.md/#core-concepts) section of the course and also goes into these topics:
  * [Scheduling](/notes/core-concepts.md/#scheduling)
  * [Logging & Monitoring](/notes/core-concepts.md/#logging-and-monitoring)
  * [Application Lifecycle Management](/notes/core-concepts.md/#application-lifecycle-management)
  * [Cluster Maintenance](/notes/core-concepts.md/#cluster-maintenance)
  * [Imperative Commands](/notes/core-concepts.md/#imperative-commands)
  * [Security](/notes/core-concepts.md/#security)
  * [Storage](/notes/core-concepts.md/#storage)
  * [Networking](/notes/core-concepts.md/#networking)
  * [Troubleshooting](/notes/core-concepts.md/#troubleshooting)
  * [JSON-PATH](/notes/core-concepts.md/#json-path)

* [definitions-cheatsheet.md](/notes/definitions-cheatsheet.md) Is a cheatsheet that has all of the yaml definition files in it for one easy-to-access place


## Review

I would **highly** recommend this course so far because of the labs and practice exams the course comes with. Just having the notes is not nearly as helpful without getting the practice from the environments that are spun up for you as well. 

The Mock Exams were a lifesaver. If you can do all of those without really thinking about it, you should be in good shape for the exam. 


### Links to Bookmark(That You Can Use On the Exam)
During this exam you can only have the tab with the exam and one other tabs that come from any of these domains/subdomains
    - [https://kubernetes.io/docs](https://kubernetes.io/docs)
    - [https://github.com/kubernetes](https://github.com/kubernetes)
    - [https://kubernetes.io/blog](https://kubernetes.io/blog)

An important thing to note is that you can only have one other tab open at a time, so I think the best strategy is to create a bookmark folder specifically for the CKA and only put links that you're allowed to click and navigate that way. Here are some links that i'll be using and bookmarking for the exam:

* [Kubernetes API Reference Doc](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.17/#daemonset-v1-apps)
  * The current kubernetes version being used on the exam(as of 2/11/2020) is v1.17 so if that changes, just update the link to whatever the newest version is instead(or make a PR :smiley:)
* [Official Kubernetes Cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)  
  * This ended up being pretty 
* [Kubectl commands docs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)
  * This link might be my go to for the exam, it seems super useful if there is a quick command you can't remember and need to enter it in quickly. 
* [Installing Kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports)
* [Upgrading a Kubeadm Cluster](https://v1-17.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
  * For when you need to use kubeadm stuff on the exam
* [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volumes)
  * These were pretty useful because I always forgot exact syntax and this way I could just copy and paste.
* [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)


### Links I Found Helpful For Reviewing(But Not Exam Useable)

These are links that aren't allowed for use on the exam, but helped with studying for the exam quite a bit. I would use these, but make sure not to get too reliant on these since you can't use them on the exam.

* [Kubernetes Imperative Commands](https://kubectl.docs.kubernetes.io/pages/imperative_porcelain/creating_resources.html)
  * This link has all the imperative commands that will be super helpful for making things fast and not having to worry about creating annoying yaml files. 
* [CKA tips medium article static pods](https://medium.com/@imarunrk/certified-kubernetes-administrator-cka-tips-and-tricks-part-2-b4f5c636eb4)
* [CKA Tips Medium Article ETCD Backup](https://medium.com/@imarunrk/certified-kubernetes-administrator-cka-tips-and-tricks-part-3-2e7b44e89a3b)
  * Both of these articles are super good for tips and learning how to do some trickier things quickly to get points and save time. The author also has other links 

## Reference
Notes from [Certified Kubernetes Administrator (CKA) with Practice Tests By Mumshad Mannambeth](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/) Udemy Course. To study for the CKA Exam

To View Exam Curriculum, go to the [cncf/curriculum repo](https://github.com/cncf/curriculum)

[CKA Certifiaction Information](https://www.cncf.io/certification/cka/)

Medium Articles Linked Written By [Arun Ramakani](https://medium.com/@imarunrk)
