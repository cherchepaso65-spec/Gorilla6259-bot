import { Bot, InlineKeyboard, Keyboard, webhookCallback } from "grammy";
import fs from "node:fs";
import path from "node:path";
import http from "node:http";
import { randomBytes } from "node:crypto";

const token = process.env.BOT_TOKEN;
const adminId = String(process.env.ADMIN_CHAT_ID || "8940384155");

if (!token) {
  throw new Error("BOT_TOKEN manquant : ajoute-le dans les variables d'environnement Render.");
}

const bot = new Bot(token);
const dataDir = process.env.DATA_DIR || "./data";
const dbFile = path.join(dataDir, "products.json");

fs.mkdirSync(dataDir, { recursive: true });
if (!fs.existsSync(dbFile)) fs.writeFileSync(dbFile, "[]", "utf8");

const sessions = new Map();
const carts = new Map();

function readProducts() {
  try {
    const data = JSON.parse(fs.readFileSync(dbFile, "utf8"));
    return Array.isArray(data) ? data : [];
  } catch (error) {
    console.error("Lecture products.json impossible :", error);
    return [];
  }
}

function saveProducts(products) {
  fs.writeFileSync(dbFile, JSON.stringify(products, null, 2), "utf8");
}

const isAdmin = (ctx) => String(ctx.from?.id || "") === adminId;

const money = (value) =>
  new Intl.NumberFormat("fr-FR", {
    style: "currency",
    currency: "EUR",
  }).format(Number(value) || 0);

function homeKeyboard() {
  return new InlineKeyboard()
    .text("🛒 Boutique", "catalog")
    .text("📲 Commander", "checkout")
    .row()
    .url("👻 Snapchat : inconnue626259", "https://www.snapchat.com/add/inconnue626259")
    .row()
    .url("👻 Snapchat : Gorilla62590", "https://www.snapchat.com/add/Gorilla62590")
    .row()
    .url("✈️ Canal Telegram", "https://t.me/+j0qkngAw9I43MGU0")
    .row()
    .text("🛍️ Mon panier", "cart")
    .text("📞 Contact", "contact");
}

function replyMenu() {
  return new Keyboard()
    .text("🛒 Boutique")
    .text("📲 Commande")
    .row()
    .text("👻 Snapchat inconnue626259")
    .text("👻 Snapchat Gorilla62590")
    .row()
    .text("✈️ Canal Telegram")
    .resized()
    .persistent();
}

async function sendHome(ctx) {
  await ctx.reply(
    "🦍 Bienvenue chez GORILLA6259 💜\n\nQualité • Sécurité • Confiance\n\nChoisis une rubrique :",
    { reply_markup: homeKeyboard() }
  );
  await ctx.reply("Menu rapide 👇", { reply_markup: replyMenu() });
}

function adminKeyboard() {
  return new InlineKeyboard()
    .text("➕ Ajouter un produit", "admin:add")
    .row()
    .text("📋 Gérer les produits", "admin:list")
    .row()
    .text("🏠 Accueil", "home");
}

async function showCatalog(ctx) {
  const products = readProducts();

  if (!products.length) {
    return ctx.reply("La boutique prépare son catalogue. Reviens bientôt 💜", {
      reply_markup: homeKeyboard(),
    });
  }

  await ctx.reply(`🛍️ Catalogue Gorilla6259 — ${products.length} produit(s)`);

  for (const product of products) {
    const caption = `💜 ${product.name}\n${product.description || ""}\n\nPrix : ${money(product.price)}`;
    const keyboard = new InlineKeyboard().text("Ajouter au panier", `add:${product.id}`);

    try {
      if (product.mediaType === "video" && product.mediaFileId) {
        await ctx.replyWithVideo(product.mediaFileId, {
          caption,
          reply_markup: keyboard,
        });
      } else if (product.mediaFileId) {
        await ctx.replyWithPhoto(product.mediaFileId, {
          caption,
          reply_markup: keyboard,
        });
      } else {
        await ctx.reply(caption, { reply_markup: keyboard });
      }
    } catch (error) {
      console.error("Envoi produit impossible :", product.id, error);
      await ctx.reply(caption, { reply_markup: keyboard });
    }
  }
}

bot.command("start", sendHome);

bot.command("help", (ctx) =>
  ctx.reply("Utilise les boutons pour consulter les produits et commander.")
);

bot.command("admin", async (ctx) => {
  if (!isAdmin(ctx)) {
    return ctx.reply("⛔ Accès réservé à l’administrateur.");
  }

  sessions.set(ctx.from.id, { step: "admin" });

  await ctx.reply("🦍 Administration Gorilla6259\nTu peux gérer les produits ici.", {
    reply_markup: adminKeyboard(),
  });
});

bot.callbackQuery("home", async (ctx) => {
  await ctx.answerCallbackQuery();
  await sendHome(ctx);
});

bot.callbackQuery("catalog", async (ctx) => {
  await ctx.answerCallbackQuery();
  await showCatalog(ctx);
});

bot.callbackQuery(/^add:(\d+)$/, async (ctx) => {
  const product = readProducts().find((item) => String(item.id) === ctx.match[1]);

  if (!product) {
    return ctx.answerCallbackQuery({ text: "Produit indisponible." });
  }

  const cart = carts.get(ctx.from.id) || [];
  cart.push({ name: product.name, price: Number(product.price) || 0 });
  carts.set(ctx.from.id, cart);

  await ctx.answerCallbackQuery({ text: "Ajouté au panier 💜" });
});

bot.callbackQuery("cart", async (ctx) => {
  await ctx.answerCallbackQuery();

  const cart = carts.get(ctx.from.id) || [];

  if (!cart.length) {
    return ctx.reply("Ton panier est vide.", { reply_markup: homeKeyboard() });
  }

  const total = cart.reduce((sum, item) => sum + item.price, 0);

  await ctx.reply(
    "🛒 TON PANIER\n" +
      cart.map((item) => `• ${item.name} : ${money(item.price)}`).join("\n") +
      `\n\nTotal : ${money(total)}`,
    {
      reply_markup: new InlineKeyboard()
        .text("📦 Valider la commande", "checkout")
        .row()
        .text("Vider le panier", "clearcart"),
    }
  );
});

bot.callbackQuery("clearcart", async (ctx) => {
  carts.delete(ctx.from.id);
  await ctx.answerCallbackQuery({ text: "Panier vidé." });
  await ctx.reply("Ton panier est vide.", { reply_markup: homeKeyboard() });
});

bot.callbackQuery("checkout", async (ctx) => {
  await ctx.answerCallbackQuery();

  const cart = carts.get(ctx.from.id) || [];

  if (!cart.length) {
    return ctx.reply("Ton panier est vide. Choisis d’abord un produit.");
  }

  sessions.set(ctx.from.id, { step: "order_name", cart });
  await ctx.reply("Pour transmettre ta commande, indique ton nom et prénom :");
});

bot.callbackQuery("contact", async (ctx) => {
  await ctx.answerCallbackQuery();

  await ctx.reply(
    "💜 Contacts :\nSnapchat : inconnue626259\nSnapchat : Gorilla62590\nCanal : https://t.me/+j0qkngAw9I43MGU0"
  );
});

bot.callbackQuery("admin:add", async (ctx) => {
  await ctx.answerCallbackQuery();
  if (!isAdmin(ctx)) return;

  sessions.set(ctx.from.id, { step: "product_media" });

  await ctx.reply(
    "➕ Ajouter un produit\n\nEnvoie maintenant une PHOTO ou une VIDÉO du produit.\nTu peux aussi envoyer /sansmedia pour ajouter le produit sans média."
  );
});

bot.callbackQuery("admin:list", async (ctx) => {
  await ctx.answerCallbackQuery();
  if (!isAdmin(ctx)) return;

  const products = readProducts();

  if (!products.length) {
    return ctx.reply("Le catalogue est vide.", { reply_markup: adminKeyboard() });
  }

  for (const product of products) {
    await ctx.reply(`${product.name} — ${money(product.price)}\nID : ${product.id}`, {
      reply_markup: new InlineKeyboard()
        .text("✏️ Modifier", `admin:edit:${product.id}`)
        .text("🗑️ Supprimer", `admin:delete:${product.id}`),
    });
  }
});

bot.callbackQuery(/^admin:edit:(\d+)$/, async (ctx) => {
  await ctx.answerCallbackQuery();
  if (!isAdmin(ctx)) return;

  const product = readProducts().find((item) => String(item.id) === ctx.match[1]);
  if (!product) return ctx.reply("Produit introuvable.");

  sessions.set(ctx.from.id, { step: "edit_product", productId: product.id });
  await ctx.reply("Envoie les nouvelles informations au format :\nNom | Prix | Description");
});

bot.callbackQuery(/^admin:delete:(\d+)$/, async (ctx) => {
  await ctx.answerCallbackQuery();
  if (!isAdmin(ctx)) return;

  const products = readProducts();
  const next = products.filter((item) => String(item.id) !== ctx.match[1]);
  saveProducts(next);

  await ctx.reply(
    next.length < products.length ? "✅ Produit supprimé du catalogue." : "Produit introuvable.",
    { reply_markup: adminKeyboard() }
  );
});

bot.on(["message:photo", "message:video"], async (ctx) => {
  if (!isAdmin(ctx)) return;

  const session = sessions.get(ctx.from.id);
  if (session?.step !== "product_media") return;

  if (ctx.message.video) {
    session.mediaType = "video";
    session.mediaFileId = ctx.message.video.file_id;
  } else {
    const photos = ctx.message.photo;
    session.mediaType = "photo";
    session.mediaFileId = photos[photos.length - 1].file_id;
  }

  session.step = "product_details";
  sessions.set(ctx.from.id, session);

  await ctx.reply(
    "✅ Média reçu !\nEnvoie maintenant : Nom | Prix | Description\nExemple : Chocolat Milka | 5.90 | Tablette 100 g"
  );
});

bot.on("message:text", async (ctx, next) => {
  const id = ctx.from.id;
  const message = ctx.message.text.trim();
  const session = sessions.get(id);

  if (message.startsWith("/")) {
    if (message === "/sansmedia" && isAdmin(ctx) && session?.step === "product_media") {
      session.mediaType = null;
      session.mediaFileId = null;
      session.step = "product_details";
      sessions.set(id, session);

      return ctx.reply(
        "Envoie : Nom | Prix | Description\nExemple : Chocolat Milka | 5.90 | Tablette 100 g"
      );
    }
    return next();
  }

  if (message === "🛒 Boutique") return showCatalog(ctx);

  if (message === "📲 Commande") {
    const cart = carts.get(id) || [];
    if (!cart.length) {
      return ctx.reply("Ton panier est vide. Choisis d’abord un produit dans Boutique 💜");
    }
    sessions.set(id, { step: "order_name", cart });
    return ctx.reply("Pour transmettre ta commande, indique ton nom et prénom :");
  }

  if (message === "👻 Snapchat inconnue626259") {
    return ctx.reply("Snapchat : inconnue626259\nhttps://www.snapchat.com/add/inconnue626259");
  }

  if (message === "👻 Snapchat Gorilla62590") {
    return ctx.reply("Snapchat : Gorilla62590\nhttps://www.snapchat.com/add/Gorilla62590");
  }

  if (message === "✈️ Canal Telegram") {
    return ctx.reply("Rejoins notre canal : https://t.me/+j0qkngAw9I43MGU0");
  }

  if (isAdmin(ctx) && session?.step === "product_details") {
    const [name, priceText, description = ""] = message.split("|").map((part) => part.trim());
    const price = Number(priceText?.replace(",", "."));

    if (!name || !priceText || !Number.isFinite(price) || price < 0) {
      return ctx.reply(
        "Format incorrect.\nEnvoie : Nom | Prix | Description\nExemple : Chocolat Milka | 5.90 | Tablette 100 g"
      );
    }

    const products = readProducts();
    products.push({
      id: Date.now(),
      name,
      price,
      description,
      mediaType: session.mediaType || null,
      mediaFileId: session.mediaFileId || null,
    });

    saveProducts(products);
    sessions.delete(id);

    return ctx.reply(
      "✅ Produit ajouté au catalogue ! Les clients peuvent maintenant le voir avec le bouton Boutique.",
      { reply_markup: adminKeyboard() }
    );
  }

  if (isAdmin(ctx) && session?.step === "edit_product") {
    const [name, priceText, description = ""] = message.split("|").map((part) => part.trim());
    const price = Number(priceText?.replace(",", "."));

    if (!name || !priceText || !Number.isFinite(price) || price < 0) {
      return ctx.reply("Format incorrect.\nEnvoie : Nom | Prix | Description");
    }

    const products = readProducts();
    const product = products.find((item) => String(item.id) === String(session.productId));

    if (!product) {
      sessions.delete(id);
      return ctx.reply("Produit introuvable.");
    }

    product.name = name;
    product.price = price;
    product.description = description;
    saveProducts(products);
    sessions.delete(id);

    return ctx.reply("✅ Produit modifié.", { reply_markup: adminKeyboard() });
  }

  if (session?.step === "order_name") {
    session.customerName = message;
    session.step = "order_phone";
    sessions.set(id, session);
    return ctx.reply("Indique ton numéro de téléphone :");
  }

  if (session?.step === "order_phone") {
    const total = session.cart.reduce((sum, item) => sum + item.price, 0);
    const order =
      `🛒 NOUVELLE COMMANDE GORILLA6259\n` +
      `Nom : ${session.customerName}\nTéléphone : ${message}\n` +
      `Client Telegram ID : ${id}\n\n` +
      session.cart.map((item) => `• ${item.name} : ${money(item.price)}`).join("\n") +
      `\n\nTOTAL : ${money(total)}`;

    await bot.api.sendMessage(adminId, order);
    sessions.delete(id);
    carts.delete(id);

    return ctx.reply(
      "✅ Ta demande de commande a été envoyée au vendeur. Il te recontactera pour confirmer.",
      { reply_markup: homeKeyboard() }
    );
  }

  await next();
});

bot.catch((error) => console.error("Erreur du bot :", error));

const port = Number(process.env.PORT || 10000);
const publicUrl = process.env.RENDER_EXTERNAL_URL;
const webhookSecret = process.env.WEBHOOK_SECRET || randomBytes(32).toString("hex");

if (publicUrl) {
  const webhookPath = `/telegram/${webhookSecret}`;
  const handleWebhook = webhookCallback(bot, "http", { secretToken: webhookSecret });

  const server = http.createServer(async (req, res) => {
    if (req.method === "GET" && req.url === "/health") {
      res.writeHead(200, { "content-type": "text/plain; charset=utf-8" });
      return res.end("Gorilla6259 bot is running");
    }

    if (req.method === "POST" && req.url === webhookPath) {
      try {
        await handleWebhook(req, res);
      } catch (error) {
        console.error("Erreur webhook Telegram :", error);
        if (!res.headersSent) {
          res.writeHead(500);
          res.end("Webhook error");
        }
      }
      return;
    }

    res.writeHead(404);
    res.end("Not found");
  });

  server.listen(port, "0.0.0.0", async () => {
    try {
      await bot.init();
      await bot.api.setWebhook(`${publicUrl}${webhookPath}`, {
        secret_token: webhookSecret,
        drop_pending_updates: false,
      });
      console.log(`Gorilla6259 webhook actif sur le port ${port}`);
    } catch (error) {
      console.error("Impossible d'activer le webhook Telegram :", error);
      process.exit(1);
    }
  });
} else {
  console.warn("RENDER_EXTERNAL_URL absent : démarrage en polling (mode local).");
  bot.start();
}
