# ALL-AMIN-KAKKO777
Online Casino script 
┌─────────────────────────────────────────────────────────────────┐
│                    ALL AMIN KAKKO 777 APP                       │
├───────────────────────────────┬─────────────────────────────────┤
│         PLAYER PORTAL         │      MASTER ADMIN HQ            │
│  - Casino Lobby & Game Hub    │  - Live Player Surveillance     │
│  - Aviator Crash Engine       │  - Game RTP & Odds Controller   │
│  - Mines 5x5 Grid Game        │  - bKash/Nagad TrxID Approvals  │
│  - European Roulette Wheel    │  - Gateway Number Settings      │
│  - 777 Gold Classic Slot      │  - User Balance & Freeze Tool   │
│  - Hi-Lo Dice Roller          │  - Financial GGR & Audit Report │
│  - bKash/Nagad Deposit Modal  │  - JSON Database Backup/Restore │
│  - Cashout & Withdrawals      │  - System Announcement Center   │
├───────────────────────────────┴─────────────────────────────────┤
│                   CENTRAL REACTIVE STATE STORE                  │
│   Users · Wallets · Transactions · Active Bets · Gateway Config │
└─────────────────────────────────────────────────────────────────┘const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
require('dotenv').config();

const app = express();
app.use(express.json());
app.use(cors());

// ১. ডাটাবেজ কানেকশন (MongoDB)
mongoose.connect(process.env.MONGO_URI || 'mongodb://localhost:27017/betting_app', {
    useNewUrlParser: true,
    useUnifiedTopology: true
})
.then(() => console.log('✅ MongoDB Connected Successfully'))
.catch(err => console.error('❌ Database Connection Error:', err));

// ২. ডাটাবেজ স্কিমা ও মডেল (ইউজার ও ট্রানজেকশন)
const UserSchema = new mongoose.Schema({
    username: { type: String, required: true, unique: true },
    password: { type: String, required: true },
    balance: { type: Number, default: 0 },
    isFrozen: { type: Boolean, default: false }
});

const TransactionSchema = new mongoose.Schema({
    userId: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
    amount: { type: Number, required: true },
    gateway: { type: String, enum: ['bKash', 'Nagad'], required: true },
    trxId: { type: String, required: true, unique: true },
    status: { type: String, enum: ['PENDING', 'APPROVED', 'REJECTED'], default: 'PENDING' },
    createdAt: { type: Date, default: Date.now }
});

const User = mongoose.model('User', UserSchema);
const Transaction = mongoose.model('Transaction', TransactionSchema);

// ==========================================
// ৩. প্লেয়ার রাউটস (Player Portal Endpoints)
// ==========================================

// ডিপোজিট রিকোয়েস্ট সাবমিট করা (bKash/Nagad)
app.post('/api/player/deposit', async (req, res) => {
    try {
        const { userId, amount, gateway, trxId } = req.body;
        
        // ইউজার স্ট্যাটাস চেক করা
        const user = await User.findById(userId);
        if (!user || user.isFrozen) {
            return res.status(403).json({ success: false, message: "অ্যাকাউন্ট ব্লক বা খুঁজে পাওয়া যায়নি।" });
        }

        const newTrx = new Transaction({
            userId,
            amount,
            gateway,
            trxId,
            status: 'PENDING'
        });

        await newTrx.save();
        res.status(201).json({ success: true, message: "ডিপোজিট রিকোয়েস্ট সফলভাবে জমা হয়েছে। অ্যাডমিন অনুমোদনের অপেক্ষায় আছে।" });
    } catch (error) {
        res.status(500).json({ success: false, error: error.message });
    }
});

// ==========================================
// ৪. মাস্টার অ্যাডমিন রাউটস (Master Admin HQ Endpoints)
// ==========================================

// পেন্ডিং ট্রানজেকশন লিস্ট দেখা
app.get('/api/admin/transactions/pending', async (req, res) => {
    try {
        const pendingTrx = await Transaction.find({ status: 'PENDING' }).populate('username balance');
        res.status(200).json({ success: true, data: pendingTrx });
    } catch (error) {
        res.status(500).json({ success: false, error: error.message });
    }
});

// অ্যাডমিন কর্তৃক TrxID অনুমোদন এবং ইউজারের ব্যালেন্স আপডেট করা
app.post('/api/admin/transaction/approve', async (req, res) => {
    try {
        const { transactionId } = req.body;

        const transaction = await Transaction.findById(transactionId);
        if (!transaction || transaction.status !== 'PENDING') {
            return res.status(400).json({ success: false, message: "ট্রানজেকশন পাওয়া যায়নি বা এটি ইতোমধ্যে প্রসেস করা হয়েছে।" });
        }

        // ট্রানজেকশন অ্যাপ্রুভ করা
        transaction.status = 'APPROVED';
        await transaction.save();

        // ইউজারের ওয়ালেট ব্যালেন্স বাড়িয়ে দেওয়া
        await User.findByIdAndUpdate(transaction.userId, {
            $inc: { balance: transaction.amount }
        });

        res.status(200).json({ success: true, message: "ট্রানজেকশন সফলভাবে অনুমোদিত হয়েছে এবং ব্যালেন্স যোগ হয়েছে।" });
    } catch (error) {
        res.status(500).json({ success: false, error: error.message });
    }
});

// সার্ভার রান করা
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
    console.log(`🚀 Server is running on port ${PORT}`);
});mkdir betting-app
cd betting-app
npm init -y
npm install express mongoose cors dotenv bcryptjs jsonwebtoken// প্লেয়ারের ডিপোজিট (bKash/Nagad) রিকোয়েস্ট হ্যান্ডেল করার একটি কোড উদাহরণ
app.post('/api/deposit', async (req, res) => {
    const { userId, amount, trxId, gateway } = req.body;
    
    // ডাটাবেজে স্টোর করা
    const newTransaction = await Transaction.create({
        userId,
        amount,
        trxId,
        gateway,
        status: 'PENDING'
    });
    
    res.status(200).json({ success: true, message: "অ্যাডমিন অনুমোদনের অপেক্ষায় আছে।" });
});npx skills add firebase/agent-skills --agent=antigravity# Firebase App Hosting

[Video](https://www.youtube.com/watch?v=saQ7Ab8ETkY)

Firebase App Hosting streamlines the development and deployment of dynamic
web apps, offering GitHub integration and integration with other Firebase
products like Authentication, Cloud Firestore, and Firebase AI Logic.
App Hosting has built-in, preconfigured support for Next.js and Angular
as well as broader support for various popular web frameworks.

[Get started](https://firebase.google.com/docs/app-hosting/get-started)

<br />


<br />


<br />

> [!IMPORTANT]
> **Looking for AI assistance?** **[Install Firebase agent skills](https://firebase.google.com/docs/ai-assistance/agent-skills)** for AI assistance with getting started and using Firebase App Hosting. Agent skills work with AI coding agents like Antigravity, Claude Code, Codex, and Cursor, etc.

## Key capabilities

|---|---|
| GitHub integration | A git commit is all that's needed to roll out a new version of your app. App Hosting can automatically deploy every time you push to a specific branch. |
| Backed by Google Cloud | App Hosting uses Google Cloud Terms of Service, and deploys your app to Google Cloud products you trust. Apps are built with Cloud Build, served on Cloud Run, and cached in Cloud CDN. Integrated services like Cloud Secret Manager keep your API keys safe. |
| Ship AI-powered features at scale | Start with AI samples that use Gemini. Protect your API endpoint's API key with Cloud Secret Manager and leverage App Hosting's streaming support to maintain fast initial load times as you add generative AI features into your app. |
| Firebase console integration | Monitor your builds and rollouts in the Firebase console so you always know what's going on. Access logs and metrics, add a custom domain, and manually trigger rollouts from the Firebase console. |

## How does it work?

1. Using the Firebase console or Firebase CLI, authorize and install the Firebase GitHub app on your repository.
2. Still in the Firebase console or Firebase CLI, create a Firebase App Hosting backend, with a repository and live branch for continuous deployment. App Hosting creates a default rollout policy for your backend to roll out to 100% of traffic immediately when a change is pushed to the target branch.
3. When a commit is pushed to your live branch, Google Cloud Developer Connect sends an event to Firebase App Hosting.
4. Responding to this event, Firebase App Hosting creates a new build for the backend connected to the repository.
   1. First, Firebase App Hosting creates a new Cloud Build build for your commit. In this job, [Google Cloud buildpacks](https://cloud.google.com/docs/buildpacks/overview) determine which framework is being used in your application to create a container and configuration (including environment variables, secrets, minimum or maximum instances, concurrency, memory, CPU, and VPC configuration) that suits your application. For more information, see [The App Hosting build process](https://firebase.google.com/docs/app-hosting/build).
   2. When the Cloud Build job is complete, your container is stored in an Artifact Registry repository dedicated to Firebase App Hosting. Firebase App Hosting then adds a new Cloud Run Revision to a Cloud Run service using your image and configuration.
5. Once your Cloud Run Revision is complete and verified healthy, Firebase App Hosting modifies its traffic configuration to point all new requests to your new Cloud Run Revision. At this point, the rollout is complete.
6. When a request is sent to a website hosted on Firebase App Hosting, the request is served by Google Cloud Load Balancer with Cloud CDN enabled. Uncached requests are sent to your Cloud Run service.

## Implementation path

|---|---|---|
|   | Set up Firebase | If you don't already have one, create a Firebase project and make sure it has the Blaze pricing plan enabled. |
|   | Set up App Hosting | With the Firebase console or the Firebase CLI, create an App Hosting backend. This is the collection of managed resources for your app, including the connection to the GitHub repository for your app. |
|   | Manage and monitor your app | When you finish creating an App Hosting backend, your app is available on its free subdomain, and you can view details about the rollout in the Firebase console. You can use the [Google Cloud console](https://console.cloud.google.com/logs/query?project=_) to view and search through your logs. |
|   | Develop your app | App Hosting automatically starts a new rollout every time a commit is pushed to your live branch. |

## Next steps

- [Get started](https://firebase.google.com/docs/app-hosting/get-started) deploying apps.
- Try an App Hosting codelab that integrates a hosted app with Firebase Authentication and Gemini API features: [Next.js](https://firebase.google.com/codelabs/firebase-nextjs) \| [Angular](https://firebase.google.com/codelabs/firebase-web).
- Learn more about the various [community-supported frameworks](https://firebaseopensource.com/platform/app_hosting) supported by App Hosting.
