#include <iostream>
#include <vector>
#include <iomanip>
#include <string>
#include <ctime>
#include <map>
#include <algorithm>
#include <locale>

using namespace std;

int generateOrderId() {
    static int id = 1000;
    return id++;
}

struct Dessert {
    string name;
    string description;
    double basePrice;
};

struct Order {
    int orderId;
    Dessert dessert;
    string size;
    double price;
    time_t timestamp;
    string username;
    bool isCancelled;
};

class DessertShop {
private:
    vector<Dessert> menu;
    vector<Order> orders;
    map<string, string> users;
    string currentUser;
    bool isLoggedIn;

public:
    DessertShop() : isLoggedIn(false) {
        menu.push_back(Dessert{"Cheesecake", "Krem peynirli klasik tat", 40.0});
        menu.push_back(Dessert{"Magnolia", "Muzlu ve bisküvili nefis tatlı", 45.0});
        menu.push_back(Dessert{"Tiramisu", "Kahve aromalı İtalyan tatlısı", 50.0});
        menu.push_back(Dessert{"Baklava", "Fıstıklı şerbetli tatlı", 35.0});
        menu.push_back(Dessert{"Profiterol", "Çikolata soslu dolgulu", 30.0});

        users["admin"] = "1234";
        users["ahmet"] = "pass1";
        users["ayse"] = "dessert";
    }

    void authMenu() {
        int choice;
        do {
            cout << "\n==== KAYIT / GİRİŞ ====\n";
            cout << "1. Kayıt Ol\n";
            cout << "2. Giriş Yap\n";
            cout << "0. Çıkış\n";
            cout << "Seçiminiz: ";
            cin >> choice;

            switch (choice) {
                case 1: registerUser(); break;
                case 2: login(); break;
                case 0:
                    cout << "Çıkış yapılıyor...\n";
                    exit(0);
                default:
                    cout << "Geçersiz seçim!\n";
            }
        } while (!isLoggedIn);
    }

    void login() {
        string username, password;
        int attempts = 3;

        while (attempts > 0) {
            cout << "\nGiriş\n";
            cout << "Kullanıcı Adı: "; cin >> username;
            cout << "Şifre: "; cin >> password;

            if (users.count(username) && users[username] == password) {
                isLoggedIn = true;
                currentUser = username;
                cout << "Giriş başarılı! Hoş geldin, " << currentUser << "!\n";
                return;
            } else {
                attempts--;
                cout << "Hatalı bilgiler. Kalan deneme: " << attempts << "\n";
            }
        }
        cout << "Giriş başarısız. Ana menüye dönülüyor...\n";
    }

    void registerUser() {
        string username, password;
        cout << "\nKayıt Ol\n";
        cout << "Yeni Kullanıcı Adı: "; cin >> username;

        if (users.count(username)) {
            cout << "Bu kullanıcı adı zaten alınmış.\n";
            return;
        }

        cout << "Şifre Belirleyin: "; cin >> password;
        users[username] = password;
        cout << "Kayıt başarılı! Şimdi giriş yapabilirsiniz.\n";
    }

    void displayMenu() {
        cout << "\n--- TATLI MENÜSÜ ---\n";
        for (size_t i = 0; i < menu.size(); ++i) {
            cout << i + 1 << ". " << menu[i].name
                 << " - " << menu[i].basePrice << " TL\n";
            cout << "   Açıklama: " << menu[i].description << "\n";
        }
    }

    double getSizeMultiplier(int sizeOption) {
        if (sizeOption == 1) return 1.0;
        if (sizeOption == 2) return 1.2;
        if (sizeOption == 3) return 1.5;
        return 1.0;
    }

    void takeOrder() {
        if (!isLoggedIn) {
            cout << "Sipariş vermek için giriş yapmalısınız.\n";
            return;
        }

        displayMenu();
        cout << "\nTatlı numarasını seçin: ";
        int choice;
        cin >> choice;
        if (choice < 1 || choice > (int)menu.size()) {
            cout << "Geçersiz seçim!\n";
            return;
        }

        double base = menu[choice - 1].basePrice;
        cout << "\nBoyutlara Göre Fiyatlar:\n";
        cout << "1. Küçük: " << fixed << setprecision(2) << base << " TL\n";
        cout << "2. Orta : " << base * getSizeMultiplier(2) << " TL\n";
        cout << "3. Büyük: " << base * getSizeMultiplier(3) << " TL\n";

        cout << "Boyut Seçin (1-Küçük, 2-Orta, 3-Büyük): ";
        int sizeChoice;
        cin >> sizeChoice;
        string sizeLabel;
        if (sizeChoice == 1) sizeLabel = "Küçük";
        else if (sizeChoice == 2) sizeLabel = "Orta";
        else if (sizeChoice == 3) sizeLabel = "Büyük";
        else {
            cout << "Geçersiz seçim!\n";
            return;
        }

        double price = base * getSizeMultiplier(sizeChoice);
        int id = generateOrderId();
        time_t now = time(0);
        Order newOrder{id, menu[choice - 1], sizeLabel, price, now, currentUser, false};
        orders.push_back(newOrder);

        cout << "Sipariş alındı: " << menu[choice - 1].name
             << " (" << sizeLabel << ") - " << fixed << setprecision(2)
             << price << " TL (No: " << id << ")\n";
    }

    void cancelOrder() {
        int id;
        cout << "\nİptal edilecek sipariş numarası: ";
        cin >> id;

        for (Order &o : orders) {
            if (o.orderId == id && o.username == currentUser && !o.isCancelled) {
                o.isCancelled = true;
                cout << "Sipariş iptal edildi.\n";
                return;
            }
        }
        cout << "Sipariş bulunamadı veya zaten iptal edilmiş.\n";
    }

    void showOrderHistory() {
        cout << "\n--- SİPARİŞ GEÇMİŞİ ---\n";
        bool found = false;
        for (const Order &o : orders) {
            if (o.username == currentUser) {
                cout << "Sipariş No: " << o.orderId
                     << " | Tatlı: " << o.dessert.name
                     << " | Boyut: " << o.size
                     << " | Fiyat: " << o.price
                     << " TL | Durum: "
                     << (o.isCancelled ? "İptal Edildi" : "Aktif") << "\n";
                found = true;
            }
        }
        if (!found) {
            cout << "Henüz siparişiniz yok.\n";
        }
    }

    void showMostPopularDessert() {
        map<string, int> dessertCounts;
        for (const Order &order : orders) {
            if (!order.isCancelled) {
                dessertCounts[order.dessert.name]++;
            }
        }

        if (dessertCounts.empty()) {
            cout << "\nHenüz yeterli sipariş yok.\n";
            return;
        }

        auto maxEntry = max_element(dessertCounts.begin(), dessertCounts.end(),
            [](const pair<string, int>& a, const pair<string, int>& b) {
                return a.second < b.second;
            });

        cout << "\n--- EN POPÜLER TATLI ---\n";
        cout << "» " << maxEntry->first << " (" << maxEntry->second << " sipariş)\n";
    }

    void showUserStatistics() {
        int totalOrders = 0;
        double totalSpent = 0.0;

        for (const Order &order : orders) {
            if (order.username == currentUser && !order.isCancelled) {
                totalOrders++;
                totalSpent += order.price;
            }
        }

        cout << "\n--- SİPARİŞ İSTATİSTİKLERİNİZ ---\n";
        cout << "Toplam Sipariş: " << totalOrders << "\n";
        cout << "Toplam Harcama: " << fixed << setprecision(2) << totalSpent << " TL\n";

        if (totalOrders > 0) {
            cout << "Ortalama Sipariş Tutarı: " << (totalSpent / totalOrders) << " TL\n";
        }
    }

    void displayMenuByPrice(bool ascending = true) {
        vector<Dessert> sortedMenu = menu;
        sort(sortedMenu.begin(), sortedMenu.end(),
            [ascending](const Dessert &a, const Dessert &b) {
                return ascending ? (a.basePrice < b.basePrice) : (a.basePrice > b.basePrice);
            });

        cout << "\n--- " << (ascending ? "UCUZDAN PAHALIYA" : "PAHALIDAN UCUZA") << " ---\n";
        for (size_t i = 0; i < sortedMenu.size(); ++i) {
            cout << i + 1 << ". " << sortedMenu[i].name
                 << " - " << sortedMenu[i].basePrice << " TL\n";
        }
    }

    void run() {
        authMenu();
        int opt;
        do {
            cout << "\n===== TATLI DÜNYASI =====\n";
            cout << "1. Menü Göster\n";
            cout << "2. Sipariş Ver\n";
            cout << "3. En Popüler Tatlı\n";
            cout << "4. Sipariş İstatistiklerim\n";
            cout << "5. Fiyata Göre Sırala\n";
            cout << "6. Sipariş İptal Et\n";
            cout << "7. Sipariş Geçmişim\n";
            cout << "0. Çıkış\n";
            cout << "Seçiminiz: ";
            cin >> opt;

            switch (opt) {
                case 1: displayMenu(); break;
                case 2: takeOrder(); break;
                case 3: showMostPopularDessert(); break;
                case 4: showUserStatistics(); break;
                case 5: {
                    int sortOpt;
                    cout << "1. Ucuzdan Pahalıya\n2. Pahalıdan Ucuza\nSeçim: ";
                    cin >> sortOpt;
                    displayMenuByPrice(sortOpt == 1);
                    break;
                }
                case 6: cancelOrder(); break;
                case 7: showOrderHistory(); break;
                case 0: cout << "Çıkış yapılıyor.\n"; break;
                default: cout << "Geçersiz seçim!\n";
            }
        } while (opt != 0);
    }
};

int main() {
    setlocale(LC_ALL, "Turkish");
    DessertShop app;
    app.run();
    return 0;
}
