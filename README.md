import os
import tkinter as tk
from tkinter import ttk

requests_path = os.path.join(os.path.dirname(__file__), 'requests')
if os.path.exists(requests_path):
    import sys
    sys.path.insert(0, requests_path)

import requests

def get_exchange_rate(api_key, base_currency, target_currency):
    url = f'https://open.er-api.com/v6/latest/{base_currency}'
    params = {'apikey': api_key}
    
    try:
        response = requests.get(url, params=params)
        data = response.json()

        if response.status_code == 200:
            exchange_rate = data['rates'].get(target_currency)
            if exchange_rate:
                return exchange_rate
            else:
                return None
        else:
            return None
    except requests.RequestException:
        return None

def currency_converter():
    try:
        amount = float(entry_amount.get())
        base_currency = combo_base.get().upper()
        target_currency = combo_target.get().upper()

        if base_currency == target_currency:
            result_var.set("Source and target currencies are the same. No conversion needed.")
        else:
            exchange_rate = get_exchange_rate(api_key, base_currency, target_currency)
            if exchange_rate is not None:
                converted_amount = amount * exchange_rate
                result_var.set(f"{amount:.2f} {base_currency} is equal to {converted_amount:.2f} {target_currency}. Exchange rate: {exchange_rate:.4f}")
            else:
                result_var.set("Unable to fetch exchange rates or unsupported target currency.")
    except ValueError:
        result_var.set("Invalid input. Please enter a numeric value for the amount.")

api_key = 'YOUR_API_KEY'

window = tk.Tk()
window.title("Currency Converter")

currency_converter_frame = ttk.Frame(window, padding="10")
currency_converter_frame.grid(column=0, row=0, sticky=(tk.W, tk.E, tk.N, tk.S))

ttk.Label(currency_converter_frame, text="Enter Amount:").grid(column=0, row=0, sticky=tk.W)
entry_amount = ttk.Entry(currency_converter_frame)
entry_amount.grid(column=1, row=0, sticky=(tk.W, tk.E))

ttk.Label(currency_converter_frame, text="Base Currency:").grid(column=0, row=1, sticky=tk.W)
combo_base = ttk.Combobox(currency_converter_frame, values=["USD", "EUR", "GBP", "JPY", "CAD", "INR"])
combo_base.grid(column=1, row=1, sticky=(tk.W, tk.E))
combo_base.current(0)

ttk.Label(currency_converter_frame, text="Target Currency:").grid(column=0, row=2, sticky=tk.W)
combo_target = ttk.Combobox(currency_converter_frame, values=["USD", "EUR", "GBP", "JPY", "CAD", "INR"])
combo_target.grid(column=1, row=2, sticky=(tk.W, tk.E))
combo_target.current(1)

ttk.Button(currency_converter_frame, text="Convert", command=currency_converter).grid(column=0, row=3, columnspan=2)

result_var = tk.StringVar()
result_label = ttk.Label(currency_converter_frame, textvariable=result_var)
result_label.grid(column=0, row=4, columnspan=2, sticky=tk.W)

window.mainloop()
