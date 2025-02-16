# pythonresimduzenleme import tkinter as tk
from tkinter import ttk, filedialog, messagebox, colorchooser
from PIL import Image, ImageTk, ImageEnhance, ImageFilter, ImageDraw, ImageFont
import os 

class ResimDuzenleyici:
    def _init_(self):
        self.pencere = tk. Tk()
        self.pencere.title("Kodlama Kampı - Resim Düzenleyici")
        self.pencere.geometry("1200*800")

        self.guncel_resim = None
        self.orjinal_resim = None
        self.dosya_yolu = None 
        self.metin_rengi = "#000000"
        self.gosterim_resmi = None

        self.parlaklik_degeri = tk.DoubleVar (value=1.0)
        self.kontrast_degeri = tk.DoubleVar(value=1.0)
        self.keskinlik_degeri = tk.DoubleVar(value=1.0)
        self.yazi_boyutu = tk.IntVar(value=24)
        self.yazi_metni = tk.StringVar(value="")
        self.yeni_genislik = tk.IntVar(value=800)
        self.yeni_yukseklik = tk.IntVar(value=600)

        self.arayuz_olustur()

     def arayuz_olustur(self):
        self.sol_panel = ttk.Frame(self.pencere, padding="10")
        self.sol_panel.pack(expand=True)

        self.resim_alani = ttk.Label(self.pencere)
        self.resim_alani.pack(expand=True)
        self.resim_alani.bind("<Button-1>", self.yazi_ekle_tikla)

        self.resim_sec_buton = ttk.Button(
            self.sol_panel,
            text="Resim Seç",
            command=self.resim_sec()
        )
        self.resim_sec_buton.pack(pady=5)

        ttk.Label(self.sol_panel, text="Parlaklık:").pack(pady=(10,0))
        self.parlaklik_slider = ttk.Scale(
            self.sol_panel,
            from_ = 0.0,
            to=2.0,
            variable=self.parlaklik_degeri,
            command=lambda x:self.efekt_uygula()
        )
        self.parlaklik_slider.pack()

        ttk.Label(self.sol_panel, text="Kontrast").pack(pady=(10.0))
        self.kontrast_slider = ttk.Scale(
            self.sol_panel,
            from=0.0,
            to=2.0,
            variable=self.kontrast_degeri,
            command=lambda x: self.efekt_uygula()
        )
        self.kontrast_slider.pack()

        ttk.Label(self.sol_panel, text="Keskimlik:").pack(pady=(10.0))
        self.keskinlik_slider = ttk.Scale(
            self.sol_panel,
            from_ = 0.0,
             to=2.0,
            variable=self.kontrast_degeri,
            command=lambda x: self.efekt_uygula()
        )
        self.keskinlik_slider.pack()

        ttk.Label(self.sol_panel, text="\nYazı EKleme:").pack()
        ttk.Scale(
            self.sol_panel,
            from_=8,
            to=72,
            variable=self.yazi_boyutu
        ).pack()

        ttk.Button(
            self.sol_panel,
            text="Yazı Rengi Seç",
            command=self.yazi_rengi_sec
        ).pack(pady=5)

        ttk.Label(self.sol_panel, text = "\nBoyutlandırma:").pack()
        ttk.Label(self.sol_panel, text= "Genişlik:").pack()
        ttk.Entry(
            self.sol_panel,
            textvariable=self.yeni_genislik
        ).pack()

        ttk.Label(self.sol_panel,text ="Yükseklik:").pack()
        ttk.Entry(
            self.sol_panel,
            textvariable=self.yeni_yukseklik
        ).pack()

        ttk.Button(
            self.sol_panel,
            text="Boyutlandır",
            command=self.resim_boyutlandir()
              ).pack(pady=5)
        
        ttk.Label(self.sol_panel,text="\nRenk Efektleri:").pack()
        ttk.Button(
            self.sol_panel,
            text="Siyah Beyaz",
            command=lambda: self.renk_efekti_uygula("siyah-beyaz")
        ).pack(pady=2)

        ttk.Button(
            self.sol_panel,
            text="Sepya",
            command=lambda: self.renk_efekti_uygula("sepya")
        ).pack(pady=2)
        

        ttk.Label(self.sol_panel, text= "\nFiltreler:").pack(pady=(10=0))


        self.kaydet_buton = ttk.Button(
            self.sol_panel,
            text= "Resmi Kaydet",
            command=self.resim_kaydet()
        )
        self.kaydet_buton.pack(pady=5)

        self.sifirla_buton= ttk.Button(
            self.sol_panel,
            text="Orjinale dön",
            command=self.reaim_sifirla()
        )
        self.sifirla_buton.pack(pady=5)

    


    def filtre_butonlari_olustur(self):
        filtreler_frame = ttk.Frame(self.sol_panel)
        filtreler_frame.pack(pady=5)

        filtreler = [
            ("Bulanık", ImageFilter. BLUR),
            ("Keskin", ImageFilter. SHARPEN)
            ("Kabartma", ImageFilter. EMBOSS)
            ("Kenar  Bulma", ImageFilter.FIND_EDGES) 
        ]

        for isim,filtre in filtreler:
            ttk.Button(
                filtreler_frame,
                text=isim
                command=lambda f=filtre: self.filtre_uygula(f)
                ).pack(pady=2)
            
            
            

    def resim_sec(self):
        self.dosya_yolu=filedialog.askopenfilename(
            filetypes=[
        ("Resim Dosyaları", "*.png *.jpg *.jpeg *.gif *.bmp")
            ]
        )
        if self.dosya_yolu:
            self.orjinal_resim= Image.open(self.dosya_yolu)
            self.guncel_resim= self.orjinal_resim.copy()
            self.resim_goster()


    def efekt_uygula(self):
        if self.orjinal_resim:
            self.guncel_resim = self.orjinal_resim.copy()
            parlaklik = ImageEnhance.Brightness(self.guncel_resim)
            self.guncel_resim = parlaklik.enhance(self.parlaklik_degeri.get())

            kontrast = ImageEnhance.Contrast(self.guncel_resim)
            self.guncel_resim = kontrast.enhance(self.kontrast_degeri.get())

            keskinlik = ImageEnhance.Sharpness(self.guncel_resim)
            self.guncel_resim = keskinlik.enhance(self.keskinlik_degeri.get())

            self.resim_goster()

    def filtre_uygula(self,filtre):
        if self.guncel_resim = self.guncel_resim.filter(filtre)
    self.resim_goster()


    def resim_sifirla(self):
        if self.orjinal_resim:
            self.parlaklik_degeri.set (1.0)
            self.kontrast_degeri.set(1.0)
            self.keskinlik_degeri.set(1.0)
            self.guncel_resim = self.orjinal_resim.copy()
            self.resim_goster()




    def resim_kaydet(self):
        if self.guncel_resim:
            kayit_yolu = filedialog.asksaveasfilename(
                defaultextension=".png",
                filetypes=[
                    ("PNG", "*.png")
                    ("JPG", "*.jpg") 
                    ( "BMP", "*.bmp")
                    ]
                )
            if kayit_yolu:
                self.guncel_resim.save(kayit_yolu)
                messagebox.showinfo(
                    "Başarı ile kaydedildi",
                    "Resminiz başarı ile kaydedildi"
                )


        
    def resim_goster(self):
        if self.guncel_resim:
            en = 800
            boy= 600
            gosterim_resmi = self.guncel_resim.copy()
            gosterim_resmi.thumbnail((en,boy))

            foto=ImageTk.PhotoImage(gosterim_resmi)
            self.resim_alani.configure(image=foto)
            self.resim_alani.image = foto
            self.gosterim_resmi = foto


    def yazi_rengi_sec(self):
        renk = colorchooser.askcolor(title="Yazı Rengi Seç")
        if renk [1]:
            self.metin_rengi = renk[1]

    def yazi_ekle_tikla(self,event):
        if self.guncel_resim and self.yazi_metni.get ():
            gosterim_en = self.gosterim_resmi.width()   
            gosterim_boy = self.gosterim_resmi.height()

            orjinal_en, orjinal_boy = self.guncel_resim.size

            x = int(event.x * (orjinal_en / gosterim en))
            y = int(event.y * (orjinal_boy / gosterim_boy))

            draaw=ImageDraw.Draw(self.guncel_resim)
            font= ImageFont.truetype("arial.tff", self.yazi_boyutu.get() )
            draw.text((x,y), self.yazi_metni.get(), fill=self.metin_rengi, font=font)
            self.resim_goster()





    def resim_boyutlandir (self):
        if self.guncel_resim:
            yeni_boyut = (self.yeni_genislik.get(), self.yeni_yukseklik.get())
            self.guncel_resim = self.guncel_resim.resize(yeni_boyut, Image.LANCZOS)
            self.resim_goster()

    def renk_efekti_uygula (self,efekt,tipi)
        if self.guncel_resim:
            if efekt_tipi == "siyah_beyaz":      # type: ignore
                self.guncel_resim =self.guncel_resim.convert ("L")
            elif efekt_tipi == " sepya":
                gri = self.guncel_resim.convert ("L")
                self.guncel_resim = Image.merge ("RGB", [
              gri.point(lambda x: x*1.05)
              gri.point(lambda x: x*0.95)
              gri.point(lambda x: x*0.80)
              ])
                self.resim_goster()

        if __name__ == "__main__":
         uygulama = ResimDuzenleyici()  
         uygulama.pencere.mainloop()            
