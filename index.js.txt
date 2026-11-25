const express = require('express');
const admin = require('firebase-admin');

// Инициализация Firebase (ключ передадим через переменную окружения)
const serviceAccount = JSON.parse(process.env.FIREBASE_SERVICE_ACCOUNT);

admin.initializeApp({
  credential: admin.credential.cert(serviceAccount),
  databaseURL: "https://happycloud-284f3-default-rtdb.firebaseio.com" // Твой URL базы (проверь в Firebase!)
});

const app = express();
app.use(express.json());

app.post('/send-push', async (req, res) => {
    const { token, title, body } = req.body;

    if (!token || !title || !body) {
        return res.status(400).send('Missing token, title, or body');
    }

    try {
        const message = {
            notification: { title: title, body: body },
            token: token
        };

        await admin.messaging().send(message);
        res.status(200).send('Push sent successfully');
    } catch (error) {
        console.error('Error sending push:', error);
        res.status(500).send('Error sending push');
    }
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});