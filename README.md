import 'package:flutter/material.dart';

void main() {
  runApp(const KasirApp());
}

class KasirApp extends StatelessWidget {
  const KasirApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Kasir App',
      theme: ThemeData(
        useMaterial3: true,
        colorSchemeSeed: Colors.indigo,
        scaffoldBackgroundColor: const Color(0xFFF5F7FB),
      ),
      home: const KasirPage(),
    );
  }
}

class Product {
  final String name;
  final int price;

  Product({
    required this.name,
    required this.price,
  });
}

class CartItem {
  final Product product;
  int quantity;

  CartItem({
    required this.product,
    this.quantity = 1,
  });

  int get subtotal => product.price * quantity;
}

class KasirPage extends StatefulWidget {
  const KasirPage({super.key});

  @override
  State<KasirPage> createState() => _KasirPageState();
}

class _KasirPageState extends State<KasirPage> {
  final TextEditingController searchController = TextEditingController();
  final TextEditingController paymentController = TextEditingController();

  final List<Product> products = [
    Product(name: 'Nasi Goreng', price: 15000),
    Product(name: 'Mie Goreng', price: 12000),
    Product(name: 'Ayam Geprek', price: 18000),
    Product(name: 'Es Teh', price: 5000),
    Product(name: 'Es Jeruk', price: 7000),
    Product(name: 'Kopi', price: 8000),
    Product(name: 'Air Mineral', price: 4000),
    Product(name: 'Kentang Goreng', price: 10000),
  ];

  final List<CartItem> cart = [];

  String searchText = '';

  int get total {
    int result = 0;

    for (final item in cart) {
      result += item.subtotal;
    }

    return result;
  }

  int get payment {
    return int.tryParse(paymentController.text) ?? 0;
  }

  int get change {
    final result = payment - total;
    return result < 0 ? 0 : result;
  }

  List<Product> get filteredProducts {
    if (searchText.trim().isEmpty) {
      return products;
    }

    return products.where((product) {
      return product.name
          .toLowerCase()
          .contains(searchText.toLowerCase());
    }).toList();
  }

  void addToCart(Product product) {
    setState(() {
      final index = cart.indexWhere(
        (item) => item.product.name == product.name,
      );

      if (index >= 0) {
        cart[index].quantity++;
      } else {
        cart.add(CartItem(product: product));
      }
    });
  }

  void increaseQuantity(int index) {
    setState(() {
      cart[index].quantity++;
    });
  }

  void decreaseQuantity(int index) {
    setState(() {
      if (cart[index].quantity > 1) {
        cart[index].quantity--;
      } else {
        cart.removeAt(index);
      }
    });
  }

  void removeItem(int index) {
    setState(() {
      cart.removeAt(index);
    });
  }

  void clearCart() {
    setState(() {
      cart.clear();
      paymentController.clear();
    });
  }

  void processPayment() {
    if (cart.isEmpty) {
      showMessage('Keranjang masih kosong.');
      return;
    }

    if (payment < total) {
      showMessage('Uang pembayaran masih kurang.');
      return;
    }

    showDialog(
      context: context,
      builder: (context) {
        return AlertDialog(
          title: const Text('Transaksi Berhasil'),
          content: Text(
            'Total: ${formatRupiah(total)}\n'
            'Bayar: ${formatRupiah(payment)}\n'
            'Kembalian: ${formatRupiah(change)}',
          ),
          actions: [
            TextButton(
              onPressed: () {
                Navigator.pop(context);
                clearCart();
              },
              child: const Text('Selesai'),
            ),
          ],
        );
      },
    );
  }

  void showMessage(String message) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text(message),
      ),
    );
  }

  String formatRupiah(int number) {
    final text = number.toString();
    final buffer = StringBuffer();

    for (int i = 0; i < text.length; i++) {
      if (i > 0 && (text.length - i) % 3 == 0) {
        buffer.write('.');
      }
      buffer.write(text[i]);
    }

    return 'Rp ${buffer.toString()}';
  }

  @override
  void dispose() {
    searchController.dispose();
    paymentController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        backgroundColor: Colors.indigo,
        foregroundColor: Colors.white,
        title: const Row(
          children: [
            Icon(Icons.point_of_sale),
            SizedBox(width: 10),
            Text(
              'Kasir App',
              style: TextStyle(
                fontWeight: FontWeight.bold,
              ),
            ),
          ],
        ),
      ),
      body: Column(
        children: [
          // SEARCH
          Padding(
            padding: const EdgeInsets.all(16),
            child: TextField(
              controller: searchController,
              onChanged: (value) {
                setState(() {
                  searchText = value;
                });
              },
              decoration: InputDecoration(
                hintText: 'Cari produk...',
                prefixIcon: const Icon(Icons.search),
                suffixIcon: searchText.isNotEmpty
                    ? IconButton(
                        onPressed: () {
                          searchController.clear();
                          setState(() {
                            searchText = '';
                          });
                        },
                        icon: const Icon(Icons.clear),
                      )
                    : null,
                filled: true,
                fillColor: Colors.white,
                border: OutlineInputBorder(
                  borderRadius: BorderRadius.circular(14),
                  borderSide: BorderSide.none,
                ),
              ),
            ),
          ),

          // PRODUK
          Expanded(
            flex: 5,
            child: Padding(
              padding: const EdgeInsets.symmetric(horizontal: 16),
              child: GridView.builder(
                itemCount: filteredProducts.length,
                gridDelegate:
                    const SliverGridDelegateWithFixedCrossAxisCount(
                  crossAxisCount: 2,
                  crossAxisSpacing: 12,
                  mainAxisSpacing: 12,
                  childAspectRatio: 1.35,
                ),
                itemBuilder: (context, index) {
                  final product = filteredProducts[index];

                  return Card(
                    color: Colors.white,
                    elevation: 2,
                    child: InkWell(
                      borderRadius: BorderRadius.circular(12),
                      onTap: () => addToCart(product),
                      child: Padding(
                        padding: const EdgeInsets.all(12),
                        child: Column(
                          mainAxisAlignment: MainAxisAlignment.center,
                          children: [
                            const Icon(
                              Icons.fastfood,
                              size: 35,
                              color: Colors.indigo,
                            ),
                            const SizedBox(height: 8),
                            Text(
                              product.name,
                              textAlign: TextAlign.center,
                              style: const TextStyle(
                                fontWeight: FontWeight.bold,
                                fontSize: 15,
                              ),
                            ),
                            const SizedBox(height: 5),
                            Text(
                              formatRupiah(product.price),
                              style: const TextStyle(
                                color: Colors.indigo,
                                fontWeight: FontWeight.bold,
                              ),
                            ),
                          ],
                        ),
                      ),
                    ),
                  );
                },
              ),
            ),
          ),

          // KERANJANG
          Expanded(
            flex: 4,
            child: Container(
              padding: const EdgeInsets.fromLTRB(16, 10, 16, 16),
              decoration: const BoxDecoration(
                color: Colors.white,
                borderRadius: BorderRadius.vertical(
                  top: Radius.circular(24),
                ),
              ),
              child: Column(
                children: [
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      const Text(
                        'Keranjang',
                        style: TextStyle(
                          fontSize: 20,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      if (cart.isNotEmpty)
                        TextButton(
                          onPressed: clearCart,
                          child: const Text('Kosongkan'),
                        ),
                    ],
                  ),

                  // LIST KERANJANG
                  Expanded(
                    child: cart.isEmpty
                        ? const Center(
                            child: Text(
                              'Belum ada produk',
                              style: TextStyle(
                                color: Colors.grey,
                              ),
                            ),
                          )
                        : ListView.builder(
                            itemCount: cart.length,
                            itemBuilder: (context, index) {
                              final item = cart[index];

                              return ListTile(
                                contentPadding: EdgeInsets.zero,
                                title: Text(
                                  item.product.name,
                                  style: const TextStyle(
                                    fontWeight: FontWeight.bold,
                                  ),
                                ),
                                subtitle: Text(
                                  formatRupiah(item.subtotal),
                                ),
                                leading: CircleAvatar(
                                  backgroundColor: Colors.indigo.shade100,
                                  child: Text(
                                    '${item.quantity}',
                                    style: const TextStyle(
                                      color: Colors.indigo,
                                      fontWeight: FontWeight.bold,
                                    ),
                                  ),
                                ),
                                trailing: Row(
                                  mainAxisSize: MainAxisSize.min,
                                  children: [
                                    IconButton(
                                      onPressed: () {
                                        decreaseQuantity(index);
                                      },
                                      icon: const Icon(
                                        Icons.remove_circle_outline,
                                      ),
                                    ),
                                    IconButton(
                                      onPressed: () {
                                        increaseQuantity(index);
                                      },
                                      icon: const Icon(
                                        Icons.add_circle_outline,
                                      ),
                                    ),
                                    IconButton(
                                      onPressed: () {
                                        removeItem(index);
                                      },
                                      icon: const Icon(
                                        Icons.delete_outline,
                                        color: Colors.red,
                                      ),
                                    ),
                                  ],
                                ),
                              );
                            },
                          ),
                  ),

                  const Divider(),

                  // TOTAL
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      const Text(
                        'Total',
                        style: TextStyle(
                          fontSize: 18,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      Text(
                        formatRupiah(total),
                        style: const TextStyle(
                          fontSize: 20,
                          color: Colors.indigo,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                    ],
                  ),

                  const SizedBox(height: 10),

                  // PEMBAYARAN
                  TextField(
                    controller: paymentController,
                    keyboardType: TextInputType.number,
                    onChanged: (_) {
                      setState(() {});
                    },
                    decoration: InputDecoration(
                      labelText: 'Uang Pembayaran',
                      prefixText: 'Rp ',
                      prefixIcon: const Icon(Icons.payments),
                      border: OutlineInputBorder(
                        borderRadius: BorderRadius.circular(12),
                      ),
                    ),
                  ),

                  const SizedBox(height: 8),

                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      const Text('Kembalian'),
                      Text(
                        formatRupiah(change),
                        style: const TextStyle(
                          fontWeight: FontWeight.bold,
                          fontSize: 16,
                        ),
                      ),
                    ],
                  ),

                  const SizedBox(height: 10),

                  // BUTTON BAYAR
                  SizedBox(
                    width: double.infinity,
                    height: 50,
                    child: ElevatedButton.icon(
                      onPressed: processPayment,
                      icon: const Icon(Icons.check),
                      label: const Text(
                        'PROSES PEMBAYARAN',
                        style: TextStyle(
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      style: ElevatedButton.styleFrom(
                        backgroundColor: Colors.indigo,
                        foregroundColor: Colors.white,
                        shape: RoundedRectangleBorder(
                          borderRadius: BorderRadius.circular(12),
                        ),
                      ),
                    ),
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }
}
