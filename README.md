# Backend For Frontend

Backends For Frontends is a Microservice design pattern and core idea is to define different backend for each kind of front-end[1][2]. This will allow  each type of user experience to have separate backend API layer (shim). The design pattern has its own advantages and disadvantages and you have to use the design pattern by looking at the requirements properly[1][2]. 

> In this guide you will learn about using  Backend For Frontend design pattern with Ballerina.

The following are the sections available in this guide.

- [What you'll build](#what-youll-build)
- [Prerequisites](#prerequisites)
- [Implementation](#implementation)
- [Testing](#testing)
- [Deployment](#deployment)
- [Observability](#observability)

## What you’ll build
To understand how BFF works lets take a real world use case of online health care management system. A health care provider have a Desktop Application and a Mobile Application for their users to have better online experience. Once the user login to the Desktop application the home page information shown in Desktop application and Mobile application may vary. Specially the resource limitations in mobile device such as screen size, battery life and data usage cause the mobile application to show minimal viable information to the end user. So the design of the application is suppose to have two different BFFs to support each user experience. Following is the design of the BFF, here the BFF is not implementing anything new, instead it consumes existing downstream services and act as a shim to translate the required information for each user experience. Following diagram demonstrates the use case with two BFF models. 

 
//![Backend For Frontend Architecture](images/Image1.svg "Backend For Frontend Architecture")
![bff_architecture](https://user-images.githubusercontent.com/8995220/43428119-81a10502-9411-11e8-8e24-4d32e0aebd12.jpeg)

In this use case we have two applications called Desktop Application and Mobile Application. For each application there is specific backend service (BFF) called Desktop BFF and Mobile BFF respectively. These BFFs consumes set of downstream services called Appointment Management Service, Medical Record Service, Notification Management Service and Message Management Service. For the purpose of the demonstration Ballerina is used to build both BFF layer and downstream service layer. 

## Prerequisites

- [Ballerina Distribution](https://ballerina.io/learn/getting-started/)
- A Text Editor or an IDE

### Optional requirements
- Ballerina IDE plugins ([IntelliJ IDEA](https://plugins.jetbrains.com/plugin/9520-ballerina), [VSCode](https://marketplace.visualstudio.com/items?itemName=WSO2.Ballerina), [Atom](https://atom.io/packages/language-ballerina))
- [Docker](https://docs.docker.com/engine/installation/)
- [Kubernetes](https://kubernetes.io/docs/setup/)

## Implementation

> If you want to skip the basics, you can download the git repo and directly move to the "Testing" section by skipping  "Implementation" section.

### Create the project structure

Ballerina is a complete programming language that supports custom project structures. Let's use the following package structure for this guide.

```
backend-for-frontend
  └── guide
      ├── appointment-mgt
      │   ├── appointment_mgt_service.bal
      │   └── tests
      │       └── appointment_mgt_service_test.bal
      ├── medical-record-mgt
      │   ├── medical_record_mgt_service.bal
      │   └── tests
      │       └── medical_record_mgt_service_test.bal
      ├── notification-mgt
      │   ├── notification_mgt_service.bal
      │   └── tests
      │       └── notification_mgt_service_test.bal
      ├── message-mgt
      │   ├── message_mgt_service.bal
      │   └── tests
      │       └── message_mgt_service_test.bal
      ├── mobile-bff
      │   ├── mobile_bff_service.bal
      │   └── tests
      │       └── mobile_bff_service_test.bal
      └── desktop-bff
          ├── desktop_bff_service.bal
          └── tests
              └── desktop_bff_service_test.bal
```

- Create the above directories in your local machine and also create empty `.bal` files.

- Then open the terminal and navigate to `service-composition/guide` and run Ballerina project initializing toolkit.
```bash
   $ ballerina init
```

### Developing the service

Let's implement the set of downstream services first.  

Appointment Management Service (appointment_mgt_service) is a REST API developed to manage health appointments for the members. For demonstration purpose it has a in-memory map to hold appointment data. It has capability to add appointments and retrieve appointments. 

##### Skeleton code for appointment_mgt_service.bal
```ballerina
import ballerina/io;
import ballerina/http;
import ballerina/log;

endpoint http:Listener listener {
   port: 9092
};

// Appointment management is done using an in-memory map.
// Add some sample appointments to 'appointmentMap' at startup.
map<json> appointmentMap;

// RESTful service.
@http:ServiceConfig { basePath: "/appointment-mgt" }
service<http:Service> appointment_service bind listener {

   @http:ResourceConfig {
       methods: ["POST"],
       path: "/appointment"
   }
   addAppointment(endpoint client, http:Request req) {
    // implementation  
   }

   @http:ResourceConfig {
       methods: ["GET"],
       path: "/appointment/list"
   }
   getAppointments(endpoint client, http:Request req) {
    // implementation       
   }

}
```

Medical Record Management Service (medical_record_mgt_service) is a REST API developed to manage medical records for the members. For demonstration purpose it has a in-memory map to hold medical record.  It has capability to add medical records and retrieve them. 

##### Skeleton code for medical_record_mgt_service.bal
```ballerina
import ballerina/io;
import ballerina/http;
import ballerina/log;

endpoint http:Listener listener {
   port: 9093
};

// Medical Record management is done using an in-memory map.
// Add some sample Medical Records to 'medicalRecordMap' at startup.
map<json> medicalRecordMap;

// RESTful service.
@http:ServiceConfig { basePath: "/medical_records" }
service<http:Service> medical_record_service bind listener {

   @http:ResourceConfig {
       methods: ["POST"],
       path: "/medical-record"
   }
   addMedicalRecord(endpoint client, http:Request req) {
       // Implementation 
   }
 
   @http:ResourceConfig {
       methods: ["GET"],
       path: "/medical-record/list"
   }
   getMedicalRecords(endpoint client, http:Request req) {
       // Implementation 
   }
 
}

```

Notification Management Service (notification_mgt_service) is a REST API developed to manage notifications. For demonstration purpose it has a in-memory map to hold notifications.  It has capability to add notifications and retrieve them. 

##### Skeleton code for notification_mgt_service.bal
```ballerina
import ballerina/io;
import ballerina/config;
import ballerina/http;
import ballerina/log;

endpoint http:Listener listener {
   port: 9094
};

// Notification management is done using an in-memory map.
// Add some sample notifications to 'notificationMap' at startup.
map<json> notificationMap;


// RESTful service.
@http:ServiceConfig { basePath: "/notification-mgt" }
service<http:Service> notification_service bind listener {

   @http:ResourceConfig {
       methods: ["POST"],
       path: "/notification"
   }
   addNotification(endpoint client, http:Request req) {
       // Implementation 
   }


   @http:ResourceConfig {
       methods: ["GET"],
       path: "/notification/list"
   }
   getNotifications(endpoint client, http:Request req) {
       // Implementation 
   }


}

```

Message Management Service (message_mgt_service) is a REST API developed to manage messages. For demonstration purpose it has a in-memory map to hold messages.  It has capability to add messages, retrieve all messages and retrieve unread messages. 

##### Skeleton code for message_mgt_service.bal
```ballerina

import ballerina/io;
import ballerina/config;
import ballerina/http;
import ballerina/log;

endpoint http:Listener listener {
   port: 9095
};

// Message management is done using an in-memory map.
// Add some sample messages to 'messageMap' at startup.
map<json> messageMap;


// RESTful service.
@http:ServiceConfig { basePath: "/message-mgt" }
service<http:Service> message_service bind listener {

   @http:ResourceConfig {
       methods: ["POST"],
       path: "/message"
   }
   addMessage(endpoint client, http:Request req) {

      // Implementation 
   }


   @http:ResourceConfig {
       methods: ["GET"],
       path: "/message/list"
   }
   getMessages(endpoint client, http:Request req) {
       // Implementation 
   }

   @http:ResourceConfig {
       methods: ["GET"],
       path: "/unread-message/list"
   }
   getUnreadMessages(endpoint client, http:Request req) {
       // Implementation 

}

```

Let’s look into the BFF implementation now. 

Mobile BFF(mobile_bff_service) is a shim used to support Mobile user experience. In this use case, when loading mobile application home page it calls a single resource in Mobile BFF and retrieve appointments, medical records and messages. Also the mobile apps having different method of sending notifications hence home page loading does not need to involve notification management service.  This will reduce number of backend calls and help to load the home pages in much efficient way. 

##### Skeleton code for mobile-bff.bal
```ballerina

import ballerina/io;
import ballerina/http;
import ballerina/log;

endpoint http:Listener listener {
   port: 9090
};

// Notification management is done using an in-memory map.
// Add some sample notifications to 'notificationMap' at startup.
//map<json> notificationMap;

// Client endpoint to communicate with appointment management service
endpoint http:Client appointmentEP {
   url: "http://localhost:9092/appointment-mgt"
};

// Client endpoint to communicate with medical record service
endpoint http:Client medicalRecordEP {
   url: "http://localhost:9093/medical_records"
};

// Client endpoint to communicate with message management service
endpoint http:Client messageEP {
   url: "http://localhost:9095/message-mgt"
};


// RESTful service.
@http:ServiceConfig { basePath: "/mobile-bff" }
service<http:Service> mobile_bff_service bind listener {

   @http:ResourceConfig {
       methods: ["GET"],
       path: "/profile"
   }
   getUserProfile(endpoint client, http:Request req) {

       // Call Appointment API and get appointment list

       // Call Medical Record API and get medical record list

       // Call Message API and get unread message list

       // Aggregate the responses 

       // Send response to the client.
      
   }

   // This API may have more resources for other functionalities
}

```

Desktop BFF(desktop_bff_service) is a shim used to support Desktop application user experience. In this use case, when loading desktop application home page it can offered to do multiple calls to its desktop_bff_service and retrieve comparatively large amount of data as per desktop application requirements.  In this use case Desktop application will call Desktop BFF separately to retrieve appointments and medical records. Also it will call Desktop BFF to retrieve Messages and Notifications in a single call. 

##### Skeleton code for appointment_mgt_service.bal
```ballerina

import ballerina/io;
import ballerina/http;
import ballerina/log;

endpoint http:Listener listener {
   port: 9091
};

// Client endpoint to communicate with appointment management service
endpoint http:Client appointmentEP {
   url: "http://localhost:9092/appointment-mgt"
};

// Client endpoint to communicate with medical record service
endpoint http:Client medicalRecordEP {
   url: "http://localhost:9093/medical_records"
};

// Client endpoint to communicate with notification management service
endpoint http:Client notificationEP {
   url: "http://localhost:9094/notification-mgt"
};

// Client endpoint to communicate with message management service
endpoint http:Client messageEP {
   url: "http://localhost:9095/message-mgt"
};


// RESTful service.
@http:ServiceConfig { basePath: "/desktop-bff" }
service<http:Service> desktop_bff_service bind listener {

   @http:ResourceConfig {
       methods: ["GET"],
       path: "/alerts"
   }
   getAlerts(endpoint client, http:Request req) {

       // This will return all message and notifications

       // Call Notification API and get notification list

       // Call Message API and get full message list

       // Generate the response from notification and message aggregation 

       // Send response to the client.  
    }

   @http:ResourceConfig {
       methods: ["GET"],
       path: "/appointments"
   }
   getAppoinments(endpoint client, http:Request req) {
       // Call Appointment API and get appointment list

       // Generate the response
      
       // Send response to the client.
           
   }
   @http:ResourceConfig {
       methods: ["GET"],
       path: "/medical-records"
   }
   getMedicalRecords(endpoint client, http:Request req) {

       // Call Medical Record API and get medical record list
    
       // Generate the response

       // Send response to the client.
   }

   // This API may have more resources for other functionalities
}

```

## Testing

### Invoking the service

Navigate to BFF/guide and run following commands in separate terminals to start all downstream services. These commands will start appointment_mgt_service, appointment_mgt_service, notification_mgt_service and notification_mgt_service on ports 9092, 9093, 9094 and 9095 respectively. 

```bash
   $ ballerina run appointment-mgt 
```

```bash
   $ ballerina run medical-record-mgt
```

```bash
   $ ballerina run notification-mgt
```

```bash
   $ ballerina run message-mgt
```

Similarly run bellow commands to start the BFF layer services. These commands will start mobile_bff_service and desktop_bff_service on ports 9090 and 9091 respectively. 


```bash
   $ ballerina run mobile-bff 
```

```bash
   $ ballerina run desktop-bff
```

For demonstration purpose let’s add some data to downstream services. Use following command to load some appointments, medical records, notifications and messages to the services. 

```bash
   $ ballerina run sample-data-publisher --config sample-data-publisher/sample-data.toml 
```

Now we have some data loaded into the downstream services hence we can call the BFF layer to retrieve the data as per the requirement. 

Mobile application can call Mobile BFF to retrieve the user profile using a single API call. Following is a sample CURL commands. 

```bash
   $ curl -X GET http://localhost:9090/mobile-bff/profile
```

Desktop application can call Desktop BFF to render user profile using few API calls. Following are set of CURL commands which can use to invoke Desktop BFF. 

```bash
   $ curl -X GET http://localhost:9091/desktop-bff/appointments
```

```bash
   $ curl -X GET http://localhost:9091/desktop-bff/medical-records
```

```bash
   $ curl -X GET http://localhost:9091/desktop-bff/alerts
```




