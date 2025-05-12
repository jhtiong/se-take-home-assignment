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
  data() {
    return {
      nextOrderId: 1,
      pendingOrders: [],
      completedOrders: [],
      bots: [],
      processingOrders: new Set()
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
    },
    addBot() {
      const bot = {
        id: this.bots.length + 1,
        isBusy: false
      };
      this.bots.push(bot);
      this.checkAndProcessOrders();
    },
    removeBot() {
      if (this.bots.length > 0) {
        const removedBot = this.bots.pop();
        // If bot was processing an order, return it to pending
        this.checkProcessingOrders();
      }
    },
    async processOrder(bot, order) {
      bot.isBusy = true;
      this.processingOrders.add(order.id);
      
      try {
        await new Promise(resolve => setTimeout(resolve, 10000));
        
        if (this.bots.includes(bot)) { // Check if bot still exists
          this.completedOrders.push(order);
          this.pendingOrders = this.pendingOrders.filter(o => o.id !== order.id);
        } else {
          // Bot was removed, return order to pending
          this.pendingOrders = this.pendingOrders.filter(o => o.id !== order.id);
          this.pendingOrders.push(order);
        }
      } finally {
        this.processingOrders.delete(order.id);
        if (this.bots.includes(bot)) {
          bot.isBusy = false;
          this.checkAndProcessOrders();
        }
      }
    },
    checkAndProcessOrders() {
      const availableBot = this.bots.find(bot => !bot.isBusy);
      const pendingOrder = this.pendingOrders.find(
        order => !this.processingOrders.has(order.id)
      );
      
      if (availableBot && pendingOrder) {
        this.processOrder(availableBot, pendingOrder);
      }
    },
    checkProcessingOrders() {
      // Reset processing orders when bots are removed
      this.processingOrders.clear();
      this.bots.forEach(bot => {
        bot.isBusy = false;
      });
      this.checkAndProcessOrders();
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

h2 {
  margin-bottom: 15px;
  color: #333;
}
</style>
