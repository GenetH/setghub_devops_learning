# OSI (Open Systems Interconnection) model

The **OSI (Open Systems Interconnection) model** is a conceptual framework used to understand and standardize the way different networking protocols interact with each other. It divides networking into **seven layers**, each with its own specific function. Here's a quick rundown:

### 1. **Physical Layer**
   - **Function**: Deals with the physical connection between devices and the transmission of raw data (bits) over a physical medium (e.g., cables, radio frequencies).
   - **Examples**: Ethernet cables, fiber optics, electrical signals, radio waves.

### 2. **Data Link Layer**
   - **Function**: Ensures reliable transmission of data across the physical network. It organizes data into frames, handles error detection, and manages flow control.
   - **Examples**: MAC (Media Access Control) addresses, switches, Wi-Fi, Ethernet.

### 3. **Network Layer**
   - **Function**: Handles the routing of data packets across different networks. This layer determines the best path for the data to reach its destination.
   - **Examples**: IP (Internet Protocol), routers, IPv4, IPv6.

### 4. **Transport Layer**
   - **Function**: Manages end-to-end communication between hosts. It ensures complete data transfer, error correction, and flow control. Protocols here ensure data is received in the correct order and without duplication.
   - **Examples**: TCP (Transmission Control Protocol), UDP (User Datagram Protocol).

### 5. **Session Layer**
   - **Function**: Manages and controls the dialogue (session) between two computers. It establishes, manages, and terminates connections between applications.
   - **Examples**: NetBIOS, PPTP (Point-to-Point Tunneling Protocol).

### 6. **Presentation Layer**
   - **Function**: Translates data between the application layer and the network. This layer handles data encryption, compression, and formatting, ensuring that data from the application layer can be sent over the network.
   - **Examples**: SSL/TLS (encryption protocols), JPEG (data formatting), ASCII.

### 7. **Application Layer**
   - **Function**: Provides network services directly to applications. This is the layer where user interaction happens, and applications can access network resources.
   - **Examples**: HTTP (web browsing), FTP (file transfer), SMTP (email), DNS (domain name system).

Each layer serves a distinct role, and data flows down the layers from one device, through the physical network, and up the layers on the receiving device.
