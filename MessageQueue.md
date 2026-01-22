# Message Queues

#### 1. The Core Concept
* **Asynchronous Processing:** Messaging queues allow the client and server to work independently (decoupled).
* **Non-Blocking:** The client sends a request and moves on immediately. It does not have to wait for the task to finish.
* **Server Efficiency:** The server processes jobs in its own preferred order and speed, preventing crashes during traffic spikes.

#### 2. Key Features
* **Persistence:** Messages can be saved (to disk), so data isn't lost if the receiver crashes.
* **Routing:** The queue acts as a traffic director, sending messages to the correct service based on rules.
* **Task Management:** Handles complexity like retrying failed messages and distributing work.

#### 3. Abstraction
* **Simplified Architecture:** Systems can focus on high-level business logic.
* **"Black Box" Delivery:** The complex details of *how* a message gets from A to B are handled entirely by the queue, abstracting implementation details from the architect.

#### 4. Terminology & Tools
* **The "Queue":** While called a queue, the internal storage policy can vary (it doesn't strictly have to be First-In-First-Out).
* **Common Tools:**
    * **Kafka:** Often used for streaming and high throughput.
    * **RabbitMQ:** Often used for complex routing.
* **Use Cases:** * **CQRS:** Segregating command (write) and query (read) responsibilities.
    * **Event Sourcing:** tracking state changes as a log of events.

#

<div style="display: flex; flex-wrap: wrap; gap: 10px; align-items: flex-start;">
<img src="./images/messagequeue/WhatsApp Image 2026-01-23 at 12.57.24 AM.jpeg" width="400">
<img src="./images/messagequeue/WhatsApp Image 2026-01-23 at 12.57.24 AM (1).jpeg" width="400">
<img src="./images/messagequeue/WhatsApp Image 2026-01-23 at 12.57.24 AM (2).jpeg" width="400">
</div>

#

#### code

```Javascript
// 1. THE BROKER (The Queue Itself)
class MessageQueue {
    constructor() {
        this.queue = [];
        this.subscribers = [];
    }

    // Producer calls this to add a message
    publish(message) {
        this.queue.push(message);
        console.log(`[Queue] Message received: "${message}"`);
        
        // Notify the consumer if they are waiting
        this.processNext(); 
    }

    // Consumer calls this to listen for messages
    subscribe(callback) {
        this.subscribers.push(callback);
        this.processNext(); // Check if there are already pending messages
    }

    // Internal Logic: Deliver message to subscriber
    processNext() {
        if (this.queue.length > 0 && this.subscribers.length > 0) {
            const message = this.queue.shift(); // FIFO (First In, First Out)
            const consumerCallback = this.subscribers[0]; // Simple Round-Robin or Single Consumer
            
            // Simulating network delay to show async nature
            setTimeout(() => {
                consumerCallback(message);
            }, 500);
        }
    }
}

// ==========================================

// 2. THE SYSTEM IN ACTION

const myQueue = new MessageQueue();

// --- The Consumer (Worker) ---
// This represents a separate server or service
console.log("Worker Service Started...");
myQueue.subscribe((msg) => {
    console.log(`✅ [Consumer] Processing task: ${msg}`);
});

// --- The Producer (Client) ---
// This represents the user facing API
console.log("API Gateway Receiving Requests...");

// Simulating incoming traffic
myQueue.publish("User Signup: sahil@example.com");
myQueue.publish("Generate Invoice: #1023");
myQueue.publish("Send Welcome Email");
```
