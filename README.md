C:\Users\peng8.yin\Documents\Sources\wonderful_egg>git diff orgin/main fe/app.js
diff --git a/fe/app.js b/fe/app.js
index 2111d9a..8bb2972 100644
--- a/fe/app.js
+++ b/fe/app.js
@@ -45,7 +45,11 @@ document.addEventListener('DOMContentLoaded', function() {
           floorDevices: [],
           // SVG map data
           svgMap: null,
-          floorPlanChart: null
+          floorPlanChart: null,
+          // AI Chat related data
+          chatHistory: [],
+          chatInput: '',
+          aiLoading: false
         }
       },

@@ -126,11 +130,78 @@ document.addEventListener('DOMContentLoaded', function() {
     },

     methods: {
+      /**
+       * Send AI chat message
+       */
+      sendMessage() {
+        if (!this.chatInput.trim() || this.aiLoading) {
+          return;
+        }
+
+        const message = this.chatInput.trim();
+        this.chatHistory.push({
+          role: 'user',
+          content: message,
+          timestamp: new Date().toISOString()
+        });
+
+        this.chatInput = '';
+        this.aiLoading = true;
+
+        socket.emit(COMMUNICATION_EVENTS.AI_CHAT, {
+          message: message,
+          history: this.chatHistory.slice(0, -1).map(msg => ({
+            role: msg.role,
+            content: msg.content
+          }))
+        });
+
+        this.$nextTick(() => {
+          this.scrollToBottom();
+        });
+      },
+
+      /**
+       * Scroll chat messages to bottom
+       */
+      scrollToBottom() {
+        const chatMessages = this.$refs.chatMessages;
+        if (chatMessages) {
+          chatMessages.scrollTop = chatMessages.scrollHeight;
+        }
+      },
+
+      /**
+       * Format chat time
+       * @param {string} time ISO time string
+       * @returns {string} Formatted time
+       */
+      formatChatTime(time) {
+        if (!time) return '';
+        return new Date(time).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
+      },
+
       /**
        * Initialize Socket.IO connection
        */
       initSocketIO() {
         // Handle connection status
+        socket.on(COMMUNICATION_EVENTS.AI_CHAT, (data) => {
+          if (data.success) {
+            console.log('AI chat response:', data.response);
+            this.chatHistory.push({
+              role: 'assistant',
+              content: data.response.content,
+              timestamp: new Date().toISOString()
+            });
+            this.aiLoading = false;
+          } else {
+            this.error = data.message;
+            this.aiLoading = false;
+            console.error('AI chat error:', data.message);
+          }
+        });
+
         socket.on(COMMUNICATION_EVENTS.CONNECTED, () => {
           console.log('Connected to server');
           this.connectionStatus = 'online';
