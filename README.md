# -psa-vin-bot
    Bot Telegram de décodage VIN PSA
import os
import re

from telegram import Update
from telegram.ext import Application, CommandHandler, ContextTypes


TOKEN = os.environ["TELEGRAM_BOT_TOKEN"]


# ============================================================
# CODES PSA
# ============================================================

PSA_CODES = {
    "AB13": {
        "name": "Alarme antivol",
        "details": "Alarme + superverrouillage + anti-soulèvement",
    },

    "AB08": {
        "name": "Alarme antivol",
        "details": "Alarme volumétrique + périmétrique + supercondamnation",
    },

    "YD01": {
        "name": "Accès et démarrage mains libres",
        "details": "ADML / Keyless",
    },
}


# ============================================================
# VALIDATION VIN
# ============================================================

def valid_vin(vin):
    vin = vin.upper().strip()

    return (
        len(vin) == 17
        and re.fullmatch(r"[A-HJ-NPR-Z0-9]{17}", vin)
    )


# ============================================================
# FICHE
# ============================================================

def create_report(vin):

    # TEST POUR L'INSTANT
    # Nous remplacerons cette partie par la récupération
    # réelle des codes PSA.

    codes = ["AB13", "YD01"]

    alarm = any(code in codes for code in ["AB13", "AB08"])
    keyless = "YD01" in codes

    text = f"""
🚗 PSA VIN CHECKER

🔑 VIN
{vin}

━━━━━━━━━━━━━━━━━━

🚨 ALARME ANTIVOL
{"✅ OUI" if alarm else "❌ NON"}

"""

    if "AB13" in codes:
        text += "└─ AB13 : Alarme + superverrouillage + anti-soulèvement\n"

    elif "AB08" in codes:
        text += "└─ AB08 : Alarme volumétrique + périmétrique\n"

    else:
        text += "└─ Aucun code alarme connu\n"

    text += f"""
━━━━━━━━━━━━━━━━━━

🔑 ACCÈS / DÉMARRAGE MAINS LIBRES
{"✅ OUI" if keyless else "❌ NON"}

"""

    if keyless:
        text += "└─ YD01 : ADML / Keyless\n"
    else:
        text += "└─ Aucun code ADML connu\n"

    text += """
━━━━━━━━━━━━━━━━━━

ℹ️ Analyse basée sur les codes PSA connus.

⚠️ La présence d'une option ne permet pas
de connaître l'état réel ON/OFF du système
sur le véhicule.
"""

    return text


# ============================================================
# /START
# ============================================================

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):

    await update.message.reply_text(
        "👋 Bienvenue sur PSA VIN Checker !\n\n"
        "🔎 Envoie :\n\n"
        "/vin TON_VIN\n\n"
        "Exemple :\n"
        "/vin VF3XXXXXXXXXXXXXXX"
    )


# ============================================================
# /VIN
# ============================================================

async def vin_command(update: Update, context: ContextTypes.DEFAULT_TYPE):

    if not context.args:
        await update.message.reply_text(
            "❌ Il manque le VIN.\n\n"
            "Exemple :\n"
            "/vin VF3XXXXXXXXXXXXXXX"
        )
        return

    vin = context.args[0].upper().strip()

    if not valid_vin(vin):
        await update.message.reply_text(
            "❌ VIN invalide.\n\n"
            "Un VIN doit contenir 17 caractères."
        )
        return

    await update.message.reply_text(
        "🔎 Analyse du VIN en cours..."
    )

    report = create_report(vin)

    await update.message.reply_text(report)


# ============================================================
# DEMARRAGE
# ============================================================

def main():

    app = Application.builder().token(TOKEN).build()

    app.add_handler(
        CommandHandler("start", start)
    )

    app.add_handler(
        CommandHandler("vin", vin_command)
    )

    print("🤖 PSA VIN Checker démarré")

    app.run_polling()


if __name__ == "__main__":
    main()
