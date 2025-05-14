<!-- Order System Component -->
<template>
  <div class="order-system">
    <div class="controls">
      <button @click="createNormalOrder">New Normal Order</button>
      <button @click="createVIPOrder">New VIP Order</button>
      <div class="bot-controls">
        <button @click="addBot">+ Bot</button>
        <button @click="removeBot">- Bot</button>
        <span>Active Bots: {{ bots.length }}</span>
      </div>
    </div>

    <div class="orders-container">
      <div class="pending-orders">
        <h2>PENDING</h2>
        <div class="order-list">
          <div v-for="order in pendingOrders" :key="order.id" class="order-card" :class="{ 'vip': order.isVIP }">
            <span>Order #{{ order.id }}</span>
            <span class="order-type">{{ order.isVIP ? 'VIP' : 'Normal' }}</span>
            <span v-if="processingOrders.has(order.id)" class="countdown">
              Processing: {{ processingTimes.get(order.id) }}s
            </span>
          </div>
        </div>
      </div>

      <div class="completed-orders">
        <h2>COMPLETE</h2>
        <div class="order-list">
          <div v-for="order in completedOrders" :key="order.id" class="order-card" :class="{ 'vip': order.isVIP }">
            <span>Order #{{ order.id }}</span>
            <span class="order-type">{{ order.isVIP ? 'VIP' : 'Normal' }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'OrderSystem',
  data() {    return {
      nextOrderId: 1,
      pendingOrders: [],
      completedOrders: [],
      bots: [],
      processingOrders: new Set(),
      processingOrdersTimers: new Map(), // Store countdown timers
      processingTimes: new Map() // Store remaining time for each order
    }
  },
  methods: {
    createNormalOrder() {
      this.pendingOrders.push({
        id: this.nextOrderId++,
        isVIP: false
      });
      this.checkAndProcessOrders();
    },
    createVIPOrder() {
      const vipOrder = {
        id: this.nextOrderId++,
        isVIP: true
      };
      
      // Find the position after last VIP order
      const lastVIPIndex = [...this.pendingOrders].reverse()
        .findIndex(order => order.isVIP);
      
      if (lastVIPIndex === -1) {
        // No VIP orders, insert at start
        this.pendingOrders.unshift(vipOrder);
      } else {
        // Insert after last VIP order
        this.pendingOrders.splice(
          this.pendingOrders.length - lastVIPIndex,
          0,
          vipOrder
        );
      }
      
      this.checkAndProcessOrders();
    },    addBot() {
      const bot = {
        id: this.bots.length + 1,
        isBusy: false,
        processingOrderId: null
      };
      this.bots.push(bot);
      
      // Find next unprocessed order respecting VIP priority
      const vipOrder = this.pendingOrders.find(
        order => order.isVIP && !this.processingOrders.has(order.id)
      );
      const normalOrder = this.pendingOrders.find(
        order => !order.isVIP && !this.processingOrders.has(order.id)
      );
      
      // Process VIP order first if available, otherwise normal order
      const nextOrder = vipOrder || normalOrder;
      if (nextOrder) {
        this.processOrder(bot, nextOrder);
      }
    },    removeBot() {
      if (this.bots.length > 0) {
        const removedBot = this.bots.pop();
        
        // If bot was processing an order, handle it
        if (removedBot.processingOrderId !== null) {
          const orderId = removedBot.processingOrderId;
          
          // Clear timer for this specific order
          clearInterval(this.processingOrdersTimers.get(orderId));
          this.processingOrdersTimers.delete(orderId);
          this.processingTimes.delete(orderId);
          this.processingOrders.delete(orderId);
          
          // Reset bot status
          removedBot.isBusy = false;
          removedBot.processingOrderId = null;
          
          // Let other bots pick up pending orders
          this.$nextTick(() => {
            this.checkAndProcessOrders();
          });
        }
      }
    },async processOrder(bot, order) {
      // Mark bot as busy and track which order it's processing
      bot.isBusy = true;
      bot.processingOrderId = order.id;
      this.processingOrders.add(order.id);
      
      // Initialize processing time (10 seconds)
      const totalTime = 10;
      this.processingTimes.set(order.id, totalTime);
      
      // Set up countdown timer
      const timer = setInterval(() => {
        if (!this.bots.includes(bot)) {
          // Bot was removed, stop timer but keep order in pending
          clearInterval(timer);
          return;
        }
        
        const currentTime = this.processingTimes.get(order.id);
        if (currentTime > 0) {
          this.processingTimes.set(order.id, currentTime - 1);
        } else if (currentTime === 0) {
          clearInterval(timer);
        }
      }, 1000);
      
      this.processingOrdersTimers.set(order.id, timer);
        try {
        await new Promise(resolve => setTimeout(resolve, totalTime * 1000));
        
        if (this.bots.includes(bot)) { // Check if bot still exists
          // Move to completed
          this.completedOrders.push(order);
          this.pendingOrders = this.pendingOrders.filter(o => o.id !== order.id);
          
          // Clean up processing state
          clearInterval(this.processingOrdersTimers.get(order.id));
          this.processingOrdersTimers.delete(order.id);
          this.processingTimes.delete(order.id);
          this.processingOrders.delete(order.id);
          
          // Reset bot and process next order
          bot.isBusy = false;
          bot.processingOrderId = null;
          this.checkAndProcessOrders();
        }
      } catch (error) {
        console.error('Error processing order:', error);
      } finally {
        if (!this.bots.includes(bot)) {
          // Bot was removed, clean up timer only
          clearInterval(this.processingOrdersTimers.get(order.id));
          this.processingOrdersTimers.delete(order.id);
        }
      }
    },    checkAndProcessOrders() {
      const availableBot = this.bots.find(bot => !bot.isBusy);
      if (!availableBot) return;
      
      // Find next unprocessed order respecting VIP priority
      const vipOrder = this.pendingOrders.find(
        order => order.isVIP && !this.processingOrders.has(order.id)
      );
      const normalOrder = this.pendingOrders.find(
        order => !order.isVIP && !this.processingOrders.has(order.id)
      );
      
      // Process VIP order first if available, otherwise normal order
      const nextOrder = vipOrder || normalOrder;
      if (nextOrder) {
        this.processOrder(availableBot, nextOrder);
      }
    },sortPendingOrders() {
      // Sort orders to maintain VIP and normal sequences
      const vipOrders = this.pendingOrders.filter(o => o.isVIP);
      const normalOrders = this.pendingOrders.filter(o => !o.isVIP);
      this.pendingOrders = [...vipOrders, ...normalOrders];
    }
  }
}
</script>

<style scoped>
.order-system {
  padding: 20px;
  max-width: 1200px;
  margin: 0 auto;
}

.controls {
  margin-bottom: 20px;
  display: flex;
  gap: 10px;
  align-items: center;
}

.bot-controls {
  margin-left: auto;
  display: flex;
  gap: 10px;
  align-items: center;
}

button {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
  background-color: #007bff;
  color: white;
  transition: background-color 0.2s;
}

button:hover {
  background-color: #0056b3;
}

.orders-container {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
}

.pending-orders, .completed-orders {
  padding: 20px;
  border-radius: 8px;
  background-color: #f8f9fa;
}

.order-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.order-card {
  padding: 15px;
  border-radius: 6px;
  background-color: white;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.order-card.vip {
  border: 2px solid #ffd700;
  background-color: #fff9e6;
}

.order-type {
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 0.8em;
  background-color: #e9ecef;
}

.order-card.vip .order-type {
  background-color: #ffd700;
  color: #000;
}

.countdown {
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 0.9em;
  background-color: #e2e3e5;
  color: #383d41;
}

h2 {
  margin-bottom: 15px;
  color: #333;
}
</style>
