# Does the choice of Kubernetes & Docker Swarm matter?
With microservices becoming more and more popular; and specifically Docker becoming the standard virtualisation/containerisation platform, the need for a consistent orchestration platform arises. It’s hard to balance ease of use, with many features which comes from complicated different requirements people have, but some patterns are emerging which makes it easier.

First of all it’s important to split up the Cloud & Hosting world into 2 categories. Private and Public cloud. The benefits of private clouds are becoming less obvious as time goes on, as you need to maintain servers yourself, hire staff and you also need the infrastructure for outages. Public cloud saves you a bit in these cases, since the provider usually has more data centers, has backup services available, and you don’t have to hire a team to maintain the hardware.

We see Kubernetes used as a standard in Google Cloud, AWS (in North America), and services like Digital Ocean starting a Kubernetes Cluster service in beta. So while Kubernetes might be harder to install, it is more versatile, and is setup to function in a full cloud environment, and contain every component from the containers/pods themselves, to load balancers, to other cloud services such as cloud builders.

Docker Swarm takes another approach, here the most important aspects are ease of use (it’s built into docker, you avoid version compatibility hell), and the general philosophy is you do the setup yourself, and you will need some external tools like load balancers.

So the conclusion is to use Kubernetes on public cloud services, and Docker Swarm on private hardware right? Not so fast… Let’s take a deeper look.

## How does Docker Swarm & Kubernetes Work?
If you’ve never worked with Docker Swarm, or just quickly glanced over it, it’s a built-in cluster tool that comes with the default Docker package. Kubernetes is google's Orchestration version for the same thing, but made more as a “complete” solution.
In short, the way both of them work, is that you have manager nodes(or pc’s) which control workers(or slaves) which runs your containers. In docker swarm the manager delegates the work to the workers, and is also delegated work to itself (This can be disabled). On Kubernetes you usually have your cloud provider, provide you with master’s so they aren’t even in the picture.

Running a bare metal Kubernetes setup(on your own), requires you to jump through quite a few hoops, since Kubernetes usually is built up around having these modules available, and being able to just “call” on them, you will most likely need to set up replacements yourself. 
If you want a load balancer on your own Kubernetes setup, you would need a service like MetalLB, which provides load balancers for TCP traffic locally in the docker network, but you would need to route to those externally as well.

One really clever thing that Docker Swarm has implemented, is it’s floating DNS. Say you have one manager, and 7 workers. You deploy nginx on only 2 instances, and you bind them by port 8080. Now, you would have to look through each worker and guess which one it’s hosted in, right? Since Docker Swarm utilizes a floating DNS, you can actually manage to access the Nginx despite of a worker not having it hosted itself, smart right?

If you point towards any worker, or the manager itself with the IP:PORT (in this case 8080), it will delegate you to the nginx service that you’ve hosted.If you dislike this, you could even create your own load balancer within Docker Swarm, which would just be a drained worker (One which is not delegated work by the manager). 

The smartest thing about Kubernetes out of the box is automatic scaling. You can make the master node listen on CPU / Memory Usage / Network traffic, and then automatically scale up your Containers. 

If you are using a public cloud you can even automatically make it “scale” up the hardware, and rent new PC’s. Also to have mirror setups available in different parts of the world, and dynamically do it in each region. This is also available on Docker swarm with extensions, and your own integrations but then you’re asking the question if you shouldn’t have gone with Kubernetes in the first place.

## Dockerswarm Really is easy!
For fun we setup a Docker Swarm Environment using the official documentation from Docker. We setup 1 manager, and 2 workers at Vultr. Surprisingly, this only took 30 minutes or so to get running, and was fairly easy to set up, no errors or anything. Given this stage, you can now manually spawn clusters on each of your workers if you’d like to, although that’s not really a good scalable solution now, is it?

With Kubernetes you’d have scaling forced onto you in your configs from the get go, and on cloud services it would already be setup. You would however have to do a much longer setup process, and for our first Kubernetes setup, it took 3 hours and we struggled a lot with load balancers, and setting the whole system up, with Docker version mismatches, Kubernetes not wanting slaves to join the cluster, and other miscellaneous issues.

On Google Cloud, everything worked like a charm, and it took no time to set up, you do however have to pay a chunk of money for the luxury. But then the question is, how many developer hours do you have to waste on doing your own setup, until just paying Google/Amazon, and it being worth it? And on top of that you also get more services, and guarantees if server outages happens.

## What should you really be using then?
It really depends, on your use-case. We used Kubernetes for a smaller scale project, and despite it’s troubles with the setup, it worked out surprisingly well. We had a bunch of workers, and they would scale automatically given the CPU Usage on each of them. We had troubles with random crashes, which Kubernetes would allow for and just restart the service without anyone noticing, so we never noticed that we had any downtime at all. 

Question comes down to, should we have used Docker Swarm? They are very similar for our smaller use-case except for manual scaling. If you don’t feel like spending hours upon hours of research to figure out how to setup Kubernetes (with MetalLB). With 3rd party plugins you can replicate auto scaling for your private network, although if you’re a corporate, you should most likely go for Kubernetes as it comes with most larger cloud services, such as AWS or Google Cloud.

In the end it’s up to you to decide what solution you will be going with. You can always hack your way through with plugins and using API’s from hosting centers such as Vultr (They have an API to spawn servers), and replicate it through that with the help of Orbiter if you choose Docker Swarm. Although things come packed and ready to use with larger cloud services, so that you can focus on development rather than struggle setting up servers. 

One consideration that smaller companies might do, is the pricing on these cloud services.
Comparing a 5 dollar instance hosted on Vultr, compared to the 7 dollar one on Google Cloud, or paying 15-20 dollars for a Kubernetes Manager, the prices are vastly different from you doing everything yourself. It comes down to however much time you want to spend on setting things up, and how valuable your time is. 

//Potentially bare slette de 4 linier herunder? Synes det er fint at stoppe ved det ovenover
You could consider the average labor of a Full Stack Developer, or a IT Technician, which would average between 20-30 dollars an hour. Setting Kubernetes could be a 1 day thing, it could be a 1 week thing. Paying 15-20 dollars to get everything set up for you, in comparison to spending developer time, which we know is expensive!


## In Conclusion
Does it really matter? Well if you were cynical and pointed to Betteridge's law of headlines you could guess the answer would be no. But the truth is, for our project Docker Swarm would have saved us time, but learning Kubernetes was a nice experience and made picking up Docker Swarm very easy. Does it matter? Kind of...

TLDR: If you are doing smaller projects on a private cloud use Dockerswarm, if you plan on doing something bigger, use the money on a good public cloud service and Kubernetes.

## Made By(08-12-2018):
 - Chris Rosendorf
 - Viktor Kim Christiansen
 - William Pfaffe

#### Sources
 -  https://docs.docker.com/get-started/
 -  https://platform9.com/blog/kubernetes-docker-swarm-compared/
 -  https://docs.docker.com/engine/swarm/swarm-tutorial/
 -  https://medium.freecodecamp.org/learn-kubernetes-in-under-3-hours-a-detailed-guide-to-orchestrating-containers-114ff420e882
 -  https://kubernetes.io/docs/home/?path=users&persona=app-developer&level=foundational
 -  https://github.com/google/metallb
 -  https://github.com/gianarb/orbiter
