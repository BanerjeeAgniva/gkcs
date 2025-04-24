# Content Delivery Networks (CDNs)
<img width="764" alt="Screenshot 2025-04-24 at 11 41 07 AM" src="https://github.com/user-attachments/assets/dac4f433-1e61-45b0-ac05-701f42fcf0c9" />

CDNs are systems that make your application **cheaper** and **faster**.

## Basic Flow

Users want to connect to a website, say **interview.com**.  
<img width="499" alt="Screenshot 2025-04-24 at 11 31 15 AM" src="https://github.com/user-attachments/assets/54a6bc6d-d255-440b-b78b-9e2beace3570" />
1. DNS maps the domain name to an IP address.  <img width="287" alt="Screenshot 2025-04-24 at 11 33 31 AM" src="https://github.com/user-attachments/assets/789a89c2-415e-4c15-9581-21708286fb21" />
2. After resolution, we keep a server that fetches HTML pages from the resolved IP address. <img width="400" alt="Screenshot 2025-04-24 at 11 31 36 AM" src="https://github.com/user-attachments/assets/fcfbaea2-3966-41f4-860e-1e3c11c1e924" />



## Problems
<img width="734" alt="Screenshot 2025-04-24 at 11 34 28 AM" src="https://github.com/user-attachments/assets/2d0b3640-3919-4dba-95c4-d6be9e6fb091" />

1. **Location of the server**:  
   - Requests near the server are served much quicker than those far away.

2. **Central placement**:  
   - If you place the server centrally, all regions experience similar latency,  
     but it may not be ideal for anyone.

3. **Regional limitations**:  
   - There may be regulations that a server cannot send data outside its region.

## Solution → Local Servers / Local Caches
<img width="761" alt="Screenshot 2025-04-24 at 11 35 09 AM" src="https://github.com/user-attachments/assets/96c6cb18-558c-4b93-9cc9-1f3c529e2419" />

These local caches are **local servers**.  
- The caches are running a server internally.  
- They have a **file system** that can be manipulated by the **mother server**.  
- This entire solution is called a **Content Delivery Network (CDN)**.

## Benefits of CDNs

1. **Close to users**  
2. **Follows regional regulations**  
3. **Content gets updated from the mother server**

## Example of CDN

- **Amazon CloudFront**
Amazon CloudFront is a content delivery network (CDN) service provided by Amazon Web Services (AWS) that accelerates the delivery of static and dynamic web content to users globally. It does this by distributing content across a worldwide network of edge locations, which are data centers that cache content closer to end users, reducing latency and improving access speeds. 

> A CDN is like a **black box** which stores content close to all your clients worldwide.
