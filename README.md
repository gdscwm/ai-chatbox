# GDSC Spring 2025 Workshop: Building an AI Chatbot with Spring AI, React, and Docker
 
 ## Introduction
 
 Welcome! In this workshop, we will be building an AI chatbot application using Spring Boot, React, Docker, and OpenAI.
 This app will let users interact with an AI-powered chatbot, ask questions, and receive responses in real time.
 
 ## Features
 
 - **Interactive Chatbot**: Engage with a chatbot to learn more about Spring AI and how to integrate it with your Spring Boot applications.
 - **Spring Boot Backend**: A robust backend powered by Spring Boot, utilizing Spring AI for processing and generating responses using the OpenAI API.
 - **React Frontend**: A user-friendly and responsive frontend built with React, providing an intuitive interface to interact with the chatbot.
 - **Dockerized Setup**: Both the frontend and backend are containerized using Docker, allowing easy setup and deployment.
 
 - ## Demo
 
 Here’s a quick demo of how the project works:
 
 ### Image Example
 
 <img width="668" alt="Screenshot 2025-03-25 at 11 03 01 PM" src="https://github.com/user-attachments/assets/e9065ce3-cbae-44e1-86fc-0ac948e62ed2" />
 
 
 ## Tech Stack
 
 - **Backend**: 
   - Java 17
   - Spring Boot
   - Spring AI
   - OpenAI API
 - **Frontend**:
   - React
   - HTML5 & CSS3
   - Axios
   - Nginx (for serving the React app)
 - **DevOps**:
   - Docker
   - Docker Compose
 
 
  ## Prerequisites
 
 Before you begin, ensure you have the following installed on your machine:
 
 - [Docker Compose](https://docs.docker.com/compose/install/)
 - [Java 17](https://adoptopenjdk.net/) (if running the Spring Boot app locally)
 - [Node.js](https://nodejs.org/) and [npm](https://www.npmjs.com/) (if running the React app locally)
 
 
 ## Get Your OpenAI Key
 
 First, you’ll need to sign up for an [OpenAI](https://platform.openai.com/settings/organization/general) account if you don’t have one. Once signed in, you’ll be taken to the homepage.
 
 In the top right corner, click the “Dashboard” menu. On the sidebar, click "API Keys," then click the "Create new secret key" button to generate your secret key:
 <img width="665" alt="Screenshot 2025-03-25 at 11 02 29 PM" src="https://github.com/user-attachments/assets/fdf17ca3-66f7-48d4-909b-1adad2d980bd" />
 
 Copy the secret key and save it somewhere safe, as you’ll need it later to connect your app to the OpenAI API.
 
 You can go through the OpenAI API reference guide to learn more about how to call the APIs, what requests it accepts, and the responses it gives.
 
 ## Build the REST API in Spring Boot
 Let’s head over to the [spring initializer](https://start.spring.io/) to generate the boilerplate code:
 
 <img width="679" alt="Screenshot 2025-03-25 at 11 05 22 PM" src="https://github.com/user-attachments/assets/6e97197f-7662-47ba-9f3b-d2e56608d469" />
 
 You can give the group, artifact, name, description, and package you choose. We’ve used Maven as the built tool, Spring boot version 3.3.3, Jar as a packaging option, and Java version 17.
 
 Hit the generate button and the zip will be downloaded. Unzip the files and import them as a Maven project into your favourite IDE (mine is Intellij).
 
 ## Configure your OpenAI key in Spring
 You can either use the existing application.properties file or create a application.yaml file. I love working with Yaml, so created a application.yaml file where I can place all my Spring Boot configurations.
 
 Add the OpenAIKey, Model, and Temperature to your application.yaml file:
 
 ```bash
 spring:
   ai:
     openai:
       chat:
         options:
           model: "gpt-3.5-turbo"
           temperature: "0.7"
       key: "PUT YOUR OPEN_API_KEY HERE"
 ```
 A similar configuration in application.properties may look like as follows:
 
 ```bash
 spring.ai.openai.chat.options.model=gpt-3.5-turbo
 spring.ai.openai.chat.options.temperature=0.7
 spring.ai.openai.key="PUT YOUR OPEN_API_KEY HERE"
 ```
 
 ## Build the ChatController
 Let’s create a GET API with the URL /ai/chat/string and a method to handle the logic:
 
 ```bash
 @RestController
 public class ChatController {
 
     @Autowired
     private final OpenAiChatModel chatModel;
 
     @GetMapping("/ai/chat/string")
     public Flux<String> generateString(@RequestParam(value = "message", defaultValue = "Tell me a joke") String message) {
         return chatModel.stream(message);
     }
 }
 ```
 * First, we’re adding @RestController to mark the ChatController class as our spring controller
 * Then, we’re injecting the dependency for the OpenAiChatModel class. It comes out of the box as part of the Spring AI dependency we’ve used.
 * The OpenAiChatModel comes with a method stream(message) which accepts the prompt as String and returns a String response (technically it’s a Flux of String as we’ve used a Reactive version of the same method).
 * Internally, OpenAiChatModel.stream(message) will call the OpenAI API and fetch the response from there. The OpenAI call will use the configuration steps mentioned in your application.yaml file, so make sure to use a valid OpenAI key.
 * We’ve created a method to handle the GET API call, which accepts the message and returns Flux<String> as the response.
 
 ## Build, Run, and Test the REST API
 Use the maven commands to build and run the Spring Boot application:
 
 ```bash
 ./mvnw clean install spring-boot:run
 ```
 Ideally, it will run on a 8080 port unless you’ve customized the port. Make sure to keep that port free to successfully run the application.
 
 You can either use Postman or the Curl command to test your REST API:
 
 ```bash
 curl --location 'http://localhost:8080/ai/chat/string?message=How%20are%20you%3F'
 ```
 
 ## Build the ChatUI using React.js
 
 We will be making it super simple and easy for the sake of this tutorial, so pardon me if I don’t follow any React best practices.
 
 ### Create App.js to Manage the ChatUI Form
 We’ll be using useState to manage the state:
 ```bash
 const [messages, setMessages] = useState([]);
 const [input, setInput] = useState('');
 const [loading, setLoading] = useState(false);
 ```
 * messages: It will store all the messages in the chat. Each message has a text and a sender (either 'user' or 'ai').
 * input: To hold what the user is typing in the text box.
 * loading: This state is set to true while the chatbot is waiting for a response from the AI, and false when the response is received
 
 Let’s create a function handleSend and call it when the user sends a message by clicking a button or pressing Enter:
 ```bash
 const handleSend = async () => {
     if (input.trim() === '') return;
 
     const newMessage = { text: input, sender: 'user' };
     setMessages([...messages, newMessage]);
     setInput('');
     setLoading(true);
 
     try {
         const response = await axios.get('http://localhost:8080/ai/chat/string?message=' + input);
         const aiMessage = { text: response.data, sender: 'ai' };
         setMessages([...messages, newMessage, aiMessage]);
     } catch (error) {
         console.error("Error fetching AI response", error);
     } finally {
         setLoading(false);
     }
 };
 ```
 Here’s what happens step by step:
 
 * Check empty input: If the input field is empty, the function returns early (nothing is sent).
 * New message from the user: A new message is added to the messages array. This message has the text (whatever the user typed) and is marked as being sent by the 'user'.
 * Reset input: The input field is cleared after the message is sent.
 * Start loading: While waiting for the AI to respond, loading is set to true to show a loading indicator.
 * Make API request: The code is used axios to request the AI chatbot API, passing the user's message. When the response comes back, a new message from the AI is added to the chat.
 * Error handling: If there is a problem getting the AI’s response, an error is logged to the console.
 * Stop loading: Finally, the loading state is turned off.
 
 Let’s write a function to update the input state whenever the user types something in the input field:
 
 ```bash
 const handleInputChange = (e) => {
     setInput(e.target.value);
 };
 ```
 Next, let’s create a function to check if the user presses the Enter key. If they do, it calls handleSend() to send the message:
 
 ```bash
 const handleKeyPress = (e) => {
     if (e.key === 'Enter') {
         handleSend();
     }
 };
 ```
 
 Now let’s create UI elements to render the chat messages:
 
 ```bash
 {messages.map((message, index) => (
     <div key={index} className={`message-container ${message.sender}`}>
         <img
             src={message.sender === 'user' ? 'user-icon.png' : 'ai-assistant.png'}
             alt={`${message.sender} avatar`}
             className="avatar"
         />
         <div className={`message ${message.sender}`}>
             {message.text}
         </div>
     </div>
 ))}
 ```
 
 This block renders all the messages in the chat:
 
 * Mapping through messages: Each message is displayed as a div using .map().
 * Message styling: The class name of the message changes based on who the sender is (user or ai), making it clear who sent the message.
 * Avatar images: Each message shows a small avatar, with a different image for the user and the AI.
 
 At last, create a button to click the message send button:
 
 ```bash
 <button onClick={handleSend}>
     <FaPaperPlane />
 </button>
 ```
 This button triggers the handleSend() function when clicked. The icon used here is a paper plane, which is common for "send" buttons.
 
 The full Chatbot.js looks as below:
 
 ```bash
 import React, { useState } from 'react';
 import axios from 'axios';
 import { FaPaperPlane } from 'react-icons/fa';
 import './Chatbot.css';
 
 const Chatbot = () => {
     const [messages, setMessages] = useState([]);
     const [input, setInput] = useState('');
     const [loading, setLoading] = useState(false);
 
     const handleSend = async () => {
         if (input.trim() === '') return;
 
         const newMessage = { text: input, sender: 'user' };
         setMessages([...messages, newMessage]);
         setInput('');
         setLoading(true);
 
         try {
             const response = await axios.get('http://localhost:8080/ai/chat/string?message=' + input);
             const aiMessage = { text: response.data, sender: 'ai' };
             setMessages([...messages, newMessage, aiMessage]);
         } catch (error) {
             console.error("Error fetching AI response", error);
         } finally {
             setLoading(false);
         }
     };
 
     const handleInputChange = (e) => {
         setInput(e.target.value);
     };
      const handleKeyPress = (e) => {
         if (e.key === 'Enter') {
             handleSend();
         }
     };
 
     return (
         <div className="chatbot-container">
             <div className="chat-header">
                 <img src="ChatBot.png" alt="Chatbot Logo" className="chat-logo" />
                 <div className="breadcrumb">Home &gt; Chat</div>
             </div>
             <div className="chatbox">
                 {messages.map((message, index) => (
                     <div key={index} className={`message-container ${message.sender}`}>
                         <img
                             src={message.sender === 'user' ? 'user-icon.png' : 'ai-assistant.png'}
                             alt={`${message.sender} avatar`}
                             className="avatar"
                         />
                         <div className={`message ${message.sender}`}>
                             {message.text}
                         </div>
                     </div>
                 ))}
                 {loading && (
                     <div className="message-container ai">
                         <img src="ai-assistant.png" alt="AI avatar" className="avatar" />
                         <div className="message ai">...</div>
                     </div>
                   )}
             </div>
             <div className="input-container">
                 <input
                     type="text"
                     value={input}
                     onChange={handleInputChange}
                     onKeyPress={handleKeyPress}
                     placeholder="Type your message..."
                 />
                 <button onClick={handleSend}>
                     <FaPaperPlane />
                 </button>
             </div>
         </div>
     );
 };
 
 export default Chatbot;
 ```
 
 Use <Chatbot/> inside the App.js to load the Chatbot UI:
 
 ```bash
 function App() {
   return (
     <div className="App">
             <Chatbot />
         </div>
   );
 }
 ```
 
 Along with this, we’re also using CSS to make our chatbot a little more beautiful. You can refer to [App.css](https://github.com/vikasrajputin/springboot-react-docker-chatbot/blob/main/chatbot-ui/src/App.css) and [Chatbot.css](https://github.com/vikasrajputin/springboot-react-docker-chatbot/blob/main/chatbot-ui/src/Chatbot.css) for that.
 
 ## Run the Frontend
 Use the npm command to run the application:
 
 ```bash
 npm start
 ```
 This should run the frontend on the URL http://localhost:3000. The application is good to be tested now.
 
 But running the backend and frontend separately is a bit of a hassle. So let’s use Docker to make the entire build process easier.
 
 ## How to Dockerize the Application
 Let’s dockerize the entire application to help bundle and ship it anywhere hassle-free. You can install and configure Docker from the [official Docker website.](https://docs.docker.com/get-started/get-docker/)
 
 ### Dockerize the Backend
 The backend of our chatbot is built with Spring Boot, so we will create a Dockerfile that builds the Spring Boot app into an executable JAR file and runs it in a container.
 
 Let’s write the Dockerfile for it:
 
 ```bash
 # Start with an official image that has Java installed
 FROM openjdk:17-jdk-alpine
 
 # Set the working directory inside the container
 WORKDIR /app
 
 # Copy the Maven/Gradle build file and source code into the container
 COPY target/chatbot-backend.jar /app/chatbot-backend.jar
 
 # Expose the application’s port
 EXPOSE 8080
 
 # Command to run the Spring Boot app
 CMD ["java", "-jar", "chatbot-backend.jar"]
 ```
 * FROM openjdk:17-jdk-alpine: This specifies that the container should be based on a lightweight Alpine Linux image that includes JDK 17, which is needed to run Spring Boot.
 * WORKDIR /app: Sets the working directory inside the container to /app, where our application files will live.
 * COPY target/chatbot-backend.jar /app/chatbot-backend.jar: Copies the built JAR file from your local machine (usually in the target folder after building the project with Maven or Gradle) into the container.
 * EXPOSE 8080: This tells Docker that the application will listen for requests on port 8080.
 * CMD ["java", "-jar", "chatbot-backend.jar"]: This specifies the command that will run when the container starts. It runs the JAR file that launches the Spring Boot app.
 
 ### Dockerize the Frontend
 The front end of our chatbot is built using React, and we can Dockerize it by creating a Dockerfile that installs the necessary dependencies, builds the app, and serves it using a lightweight web server like NGINX.
 
 Let’s write the Dockerfile for the React frontend:
 ```bash
 # Use a Node image to build the React app
 FROM node:16-alpine AS build
 
 # Set the working directory inside the container
 WORKDIR /app
 
 # Copy the package.json and install the dependencies
 COPY package.json package-lock.json ./
 RUN npm install
 
 # Copy the rest of the application code and build it
 COPY . .
 RUN npm run build
 
 # Use a lightweight NGINX server to serve the built app
 FROM nginx:alpine
 COPY --from=build /app/build /usr/share/nginx/html
 
 # Expose port 80 for the web traffic
 EXPOSE 80
 
 # Start NGINX
 CMD ["nginx", "-g", "daemon off;"]
 ```
 * FROM node:16-alpine AS build: This uses a lightweight Node.js image to build the React app. We install all dependencies and build the app inside this container.
 * WORKDIR /app: Sets the working directory inside the container to /app.
 * COPY package.json package-lock.json ./: Copies package.json and package-lock.json to install dependencies.
 * RUN npm install: Installs the dependencies listed in the package.json.
 * COPY . .: Copies all the frontend source code into the container.
 * RUN npm run build: Builds the React application. The built files will be in a build folder.
 * FROM nginx:alpine: After building the app, this line starts a new container based on the nginx web server.
 * COPY --from=build /app/build /usr/share/nginx/html: Copies the built React app from the first container into the nginx container, placing it in the default folder where NGINX serves files.
 * EXPOSE 80: This exposes port 80, which NGINX uses to serve web traffic.
 * CMD ["nginx", "-g", "daemon off;"]: This starts the NGINX server in the foreground to serve your React app.
 
 ### Docker Compose to Run Both
 Now that we have separate Dockerfiles for the frontend and backend, we’ll use docker-compose to orchestrate running both containers at once.
 
 Let’s write the docker-compose.yml file inside the root directory of the project:
 
 ```bash
 version: '3'
 services:
   backend:
     build: ./backend
     ports:
       - "8080:8080"
     networks:
       - chatbot-network
 
   frontend:
     build: ./frontend
     ports:
       - "3000:80"
     depends_on:
       - backend
     networks:
       - chatbot-network
 
 networks:
   chatbot-network:
     driver: bridge
 ```
 * version: '3': This defines the version of Docker Compose being used.
 * services:: This defines the services we want to run.
     * backend: This service builds the backend using the Dockerfile located in the ./backend directory and exposes port 8080.
     * frontend: This service builds the front end using the Dockerfile located in the ./frontend directory. It maps port 3000 on the host to port 80 inside the container.
 * depends_on:: This makes sure the front end waits for the backend to be ready before it starts.
 * networks:: This section defines a shared network so that both the backend and frontend can communicate with each other.
 
 
 ## Run the Application
 
 ```bash
 docker-compose up --build
 ```
 
 This command will:
  * Build both the frontend and backend images.
  * Start both containers (backend on port 8080, frontend on port 3000).
  * Set up networking so that both services can communicate.
 
 Now, you can head over to http://localhost:3000 load the Chatbot UI and start asking your questions to the AI.
 
 
 ## Congratulations :tada:
 You’ve successfully built a full-stack chatbot application using Spring Boot, React, Docker, and OpenAI.
 
 ## Acknowledgments
 Thanks to FreeCodeCamp for providing the template to this workshop [here!](https://www.freecodecamp.org/news/ai-chatbot-with-spring-react-docker/#heading-prerequisites)
 
 You can visit the original project repo [here.](https://github.com/vikasrajputin/springboot-react-docker-chatbot/edit/main/README.md)
 
