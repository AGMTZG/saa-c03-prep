# 🧠 EC2 Purchasing Options = Problems They Solve

## 🟢 On-Demand  
> ❓ *“I don’t know what will happen”*

- Unpredictable traffic  
- Short-term usage  
- Need instant start/stop  

✅ **Solution:** maximum flexibility  
❌ **Tradeoff:** expensive  

---

## 🔵 Reserved Instances (RI)  
> 📊 *“I know exactly what I’ll use”*

- Steady workload (e.g. database)  
- Long-running systems  
- Predictable capacity  

✅ **Solution:** big discount for committing  
❌ **Tradeoff:** low flexibility  

---

## 🟣 Savings Plans  
> 🔄 *“I know I’ll use compute, but not exactly how”*

- Predictable usage but changing configs  
- Might switch instance types  

✅ **Solution:** discount + flexibility  
❌ **Tradeoff:** time commitment  

---

## 🟡 Spot Instances  
> 💸 *“I want the cheapest compute possible”*

- Can tolerate interruptions  
- Distributed / fault-tolerant systems  

✅ **Solution:** massive cost savings  
❌ **Tradeoff:** can be terminated anytime  

---

## 🔴 Dedicated Hosts  
> ⚖️ *“I have licensing or compliance constraints”*

- BYOL (per-core, per-socket licenses)  
- Regulatory requirements  

✅ **Solution:** full physical server control  
❌ **Tradeoff:** very expensive  

---

## 🟠 Dedicated Instances  
> 🔐 *“I need isolation, but not full hardware control”*

- No shared hardware with other customers  

✅ **Solution:** tenant isolation  
❌ **Tradeoff:** less control than hosts  

---

## 🟤 Capacity Reservations  
> 🚨 *“I MUST have capacity available”*

- Critical workloads  
- Need guaranteed capacity in AZ  

✅ **Solution:** capacity guarantee  
❌ **Tradeoff:** no discount, pay even if unused  

---

# ⚡ Ultra-Compact Mental Model

- Don’t know usage → **On-Demand**  
- Know usage → **Reserved / Savings Plans**  
- Want cheapest → **Spot**  
- Need compliance → **Dedicated Hosts**  
- Need isolation → **Dedicated Instances**  
- Need guaranteed capacity → **Capacity Reservations**  

---

# 🧪 Quick Decision Framework

Ask yourself:

1. Is usage predictable?  
2. Can the workload be interrupted?  
3. Do I need guaranteed capacity or compliance?  
