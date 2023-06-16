---
title: Private Networking
---

<PriorityBoardingBanner />


Private Networking is a feature within Railway that will open network communication through a IPv6 wireguard mesh only accessible to your Railway services within an environment. 

<Image src="https://res.cloudinary.com/railway/image/upload/v1686946888/docs/CleanShot_2023-06-16_at_16.21.08_2x_lgp9ne.png"
alt="Preview of What The Guide is Building"
layout="intrinsic"
width={1310} height={420} quality={100} />

This will allow you to have a private network between your services, helpful for situations where you want to have a public gateway for your API but leave internal communication private.

## Enabling Private Networking

All new projects have private networking enabled and services will get a new DNS name under the `railway.internal` domain. This DNS name will resolve to the internal IPv6 address of the services within a project.

<Image src="https://res.cloudinary.com/railway/image/upload/v1686946842/docs/CleanShot_2023-06-16_at_16.15.35_2x_woehyq.png"
alt="Preview of What The Guide is Building"
layout="intrinsic"
width={1442} height={510} quality={100} />

For environments created before 2023/06/16 - you can enable it on the service settings page for old projects. This will enable private networking and service discovery for all services within the environment.

To communicate on the private network, you must bind to a IPv6 port and use the internal DNS name of the service. For example, if you have a service called `api` and you want to communicate with it from another service, you would use `api.railway.internal` as the hostname.

If you wish to open a service that has a public, you can use the `PORT` environment variable to specify the public port. This will allow Railway to route traffic to the public port. 

## How it works

Every service within an environment now has its own subnet within an environment. Under the hood, Railway is using encrypted Wireguard tunnels to create a mesh network between all services within an environment. This allows Railway to route traffic between services without having to expose any ports publicly. **Note: You cannot use private networking to communicate with services in other environments.**

Every service gets a DNS name under the `railway.internal` domain. This DNS name will resolve to the internal IP address of the service. This allows you to communicate with any service within an environment without having to expose any ports publicly. Any valid IPv6 traffic is allowed, UDP, TCP and HTTP. 

## Communicating with services

To communicate with a service, you must bind to a IPv6 port. On most web frameworks, you can do this via `::` and specifying the port(s) you want to bind to.

To communicate with a service, you must use the internal DNS name of the service.

### Internal DNS

Railway uses the `*.railway.internal` domain for all internal DNS names. This domain will resolve to the internal IP address of the service. Within the service settings you can change the service name you can refer to but not the internal DNS root. Ex. `api-1.railway.internal` -> `api-2.railway.internal`

Requests to replica DNS service address will be round robin'd between all replicas.

## Caveats

During the feature development process we found a few caveats that you should be aware of.

- Railway databases are not accessible via the private network, we are moving towards a system where DBs are services with volumes attached. 
- You will need to establish a wireguard tunnel to external services if you wish to vendor requests in your application.
- You will need to bind to a IPv6 port to receive traffic on the private network.
- You will need to make a new project and a new environment to use private networking.
- Private take 100ms to initialize, we ask that you set initial private requests on a retry loop. 
- We don't support IPv4 private networking 