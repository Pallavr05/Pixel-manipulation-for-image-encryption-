# Pixel-manipulation-for-image-encryption-
import tkinter as tk
from tkinter import filedialog, messagebox
from PIL import Image, ImageTk
import random
import pickle
import os

class ImageEncryptorApp:
    def __init__(self, root):
        self.root = root
        self.root.title("🖼️ Image Encryptor & Decryptor")
        self.image_path = ""
        self.key = 0
        self.key_file = "shuffle_key.pkl"

        self.setup_gui()

    def setup_gui(self):
        self.label = tk.Label(self.root, text="Select an image file to encrypt or decrypt")
        self.label.pack(pady=10)

        self.canvas = tk.Label(self.root)
        self.canvas.pack(pady=10)

        self.key_entry = tk.Entry(self.root)
        self.key_entry.pack(pady=5)
        self.key_entry.insert(0, "Enter key (e.g. 123)")

        self.upload_btn = tk.Button(self.root, text="📂 Browse Image", command=self.load_image)
        self.upload_btn.pack(pady=5)

        self.encrypt_btn = tk.Button(self.root, text="🔐 Encrypt", command=self.encrypt)
        self.encrypt_btn.pack(pady=5)

        self.decrypt_btn = tk.Button(self.root, text="🔓 Decrypt", command=self.decrypt)
        self.decrypt_btn.pack(pady=5)

    def load_image(self):
        path = filedialog.askopenfilename(filetypes=[("Image files", "*.jpg *.png *.bmp")])
        if path:
            self.image_path = path
            img = Image.open(path)
            img.thumbnail((300, 300))
            img_tk = ImageTk.PhotoImage(img)
            self.canvas.configure(image=img_tk)
            self.canvas.image = img_tk

    def encrypt(self):
        if not self.image_path or not self.key_entry.get().isdigit():
            messagebox.showerror("Error", "Please select an image and enter a valid key.")
            return

        key = int(self.key_entry.get())
        output_path = "encrypted_image.png"

        image = Image.open(self.image_path).convert("RGB")
        pixels = list(image.getdata())
        width, height = image.size

        random.seed(key)
        modified = [((r + key) % 256, (g + key) % 256, (b + key) % 256) for r, g, b in pixels]

        indices = list(range(len(modified)))
        random.shuffle(indices)
        shuffled = [modified[i] for i in indices]

        with open(self.key_file, "wb") as f:
            pickle.dump(indices, f)

        encrypted_img = Image.new("RGB", (width, height))
        encrypted_img.putdata(shuffled)
        encrypted_img.save(output_path)

        messagebox.showinfo("Success", f"Encrypted image saved as {output_path}.")

    def decrypt(self):
        if not self.key_entry.get().isdigit():
            messagebox.showerror("Error", "Please enter a valid key.")
            return

        try:
            with open(self.key_file, "rb") as f:
                indices = pickle.load(f)
        except FileNotFoundError:
            messagebox.showerror("Error", "Key file not found.")
            return

        key = int(self.key_entry.get())
        path = filedialog.askopenfilename(title="Select Encrypted Image", filetypes=[("Image files", "*.png")])
        if not path:
            return

        image = Image.open(path).convert("RGB")
        pixels = list(image.getdata())
        width, height = image.size

        unshuffled = [None] * len(pixels)
        for i, idx in enumerate(indices):
            unshuffled[idx] = pixels[i]

        decrypted = [((r - key) % 256, (g - key) % 256, (b - key) % 256) for r, g, b in unshuffled]

        decrypted_img = Image.new("RGB", (width, height))
        decrypted_img.putdata(decrypted)
        decrypted_img.save("decrypted_image.png")

        messagebox.showinfo("Success", "Decrypted image saved as decrypted_image.png.")


# Run the app
if __name__ == "__main__":
    root = tk.Tk()
    app = ImageEncryptorApp(root)
    root.mainloop()
