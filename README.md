import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Shopping Cart',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        primarySwatch: Colors.blue,
        scaffoldBackgroundColor: const Color(0xFFF5F6F8),
      ),
      home: const ShoppingCartPage(),
    );
  }
}

class Product {
  final String id;
  final String name;
  final String brand;
  final int price;
  final String imageUrl;
  int likes;
  int quantity;
  bool isSelected;

  Product({
    required this.id,
    required this.name,
    required this.brand,
    required this.price,
    required this.imageUrl,
    required this.likes,
    this.quantity = 1,
    this.isSelected = false,
  });
}

class ShoppingCartPage extends StatefulWidget {
  const ShoppingCartPage({super.key});

  @override
  State<ShoppingCartPage> createState() => _ShoppingCartPageState();
}

class _ShoppingCartPageState extends State<ShoppingCartPage> {
  int _selectedIndex = 2; // Index Keranjang
  bool _showBanner = false;
  String _bannerText = '';

  final List<Product> _products = [
    Product(
      id: '1',
      name: 'Smartwatch Series 8',
      brand: 'Apple',
      price: 4500000,
      likes: 18,
      imageUrl: 'https://images.unsplash.com/photo-1546868871-7041f2a55e12?w=300',
    ),
    Product(
      id: '2',
      name: 'Mechanical Keyboard',
      brand: 'Keychron K2',
      price: 1250000,
      likes: 24,
      imageUrl: 'https://images.unsplash.com/photo-1587829741301-dc798b83add3?w=300',
    ),
    Product(
      id: '3',
      name: 'Backpack Laptop 20L',
      brand: 'Eiger',
      price: 480000,
      likes: 9,
      imageUrl: 'https://images.unsplash.com/photo-1553062407-98eeb64c6a62?w=300',
    ),
  ];

  int get _totalItems => _products.fold(0, (sum, item) => sum + item.quantity);

  int get _totalPrice => _products.fold(0, (sum, item) => sum + (item.price * item.quantity));

  void _toggleSelect(int index) {
    setState(() {
      _products[index].isSelected = !_products[index].isSelected;
    });
  }

  void _incrementLike(int index) {
    setState(() {
      _products[index].likes++;
    });
  }

  void _showLongPressInfo(int index) {
    setState(() {
      _showBanner = true;
      _bannerText = '${_products[index].name} ${_products[index].brand} telah dipilih.';
    });
  }

  void _updateQuantity(int index, int delta) {
    setState(() {
      if (_products[index].quantity + delta >= 1) {
        _products[index].quantity += delta;
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    final mediaQuery = MediaQuery.of(context);
    final isWideScreen = mediaQuery.size.width > 600;

    return Scaffold(
      appBar: AppBar(
        backgroundColor: const Color(0xFF0D6EFD),
        elevation: 0,
        title: const Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              'My Cart',
              style: TextStyle(fontWeight: FontWeight.bold, fontSize: 18, color: Colors.white),
            ),
            Text(
              'Belanja lebih mudah setiap hari',
              style: TextStyle(fontSize: 11, color: Colors.white70),
            ),
          ],
        ),
        actions: [
          IconButton(
            icon: const Icon(Icons.search, color: Colors.white),
            onPressed: () {},
          ),
        ],
      ),
      body: Center(
        child: Container(
          constraints: BoxConstraints(
            maxWidth: isWideScreen ? 600 : double.infinity,
          ),
          child: Column(
            children: [
              if (_showBanner)
                Container(
                  margin: const EdgeInsets.all(12),
                  padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 10),
                  decoration: BoxDecoration(
                    color: const Color(0xFF198754),
                    borderRadius: BorderRadius.circular(8),
                  ),
                  child: Row(
                    children: [
                      const Icon(Icons.check_circle, color: Colors.white, size: 20),
                      const SizedBox(width: 10),
                      Expanded(
                        child: Column(
                          crossAxisAlignment: CrossAxisAlignment.start,
                          children: [
                            const Text(
                              'Produk dipilih!',
                              style: TextStyle(color: Colors.white, fontWeight: FontWeight.bold, fontSize: 12),
                            ),
                            Text(
                              _bannerText,
                              style: const TextStyle(color: Colors.white70, fontSize: 11),
                            ),
                          ],
                        ),
                      ),
                      GestureDetector(
                        onTap: () => setState(() => _showBanner = false),
                        child: const Icon(Icons.close, color: Colors.white, size: 18),
                      )
                    ],
                  ),
                ),

              Expanded(
                child: ListView.builder(
                  padding: const EdgeInsets.all(16),
                  itemCount: _products.length,
                  itemBuilder: (context, index) {
                    final product = _products[index];
                    return ProductCard(
                      product: product,
                      onTap: () => _toggleSelect(index),
                      onDoubleTap: () => _incrementLike(index),
                      onLongPress: () => _showLongPressInfo(index),
                      onIncrement: () => _updateQuantity(index, 1),
                      onDecrement: () => _updateQuantity(index, -1),
                    );
                  },
                ),
              ),

              _buildCartSummary(),
            ],
          ),
        ),
      ),
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _selectedIndex,
        onTap: (idx) => setState(() => _selectedIndex = idx),
        type: BottomNavigationBarType.fixed,
        selectedItemColor: const Color(0xFF0D6EFD),
        unselectedItemColor: Colors.grey,
        items: const [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Beranda'),
          BottomNavigationBarItem(icon: Icon(Icons.grid_view), label: 'Kategori'),
          BottomNavigationBarItem(icon: Icon(Icons.shopping_cart), label: 'Keranjang'),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Akun'),
        ],
      ),
    );
  }

  Widget _buildCartSummary() {
    String formattedPrice = _totalPrice.toString().replaceAllMapped(
      RegExp(r'(\d{1,3})(?=(\d{3})+(?!\d))'),
      (Match m) => '${m[1]}.',
    );

    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 20, vertical: 14),
      decoration: BoxDecoration(
        color: Colors.white,
        boxShadow: [
          BoxShadow(
            color: Colors.black.withOpacity(0.05),
            blurRadius: 10,
            offset: const Offset(0, -5),
          ),
        ],
      ),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Text(
                'Total ($_totalItems produk)',
                style: const TextStyle(fontSize: 12, color: Colors.grey),
              ),
              const SizedBox(height: 2),
              Text(
                'Rp $formattedPrice',
                style: const TextStyle(
                  fontSize: 16,
                  fontWeight: FontWeight.bold,
                  color: Color(0xFF0D6EFD),
                ),
              ),
            ],
          ),
          ElevatedButton(
            onPressed: () {},
            style: ElevatedButton.styleFrom(
              backgroundColor: const Color(0xFF0D6EFD),
              padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 12),
              shape: RoundedRectangleBorder(
                borderRadius: BorderRadius.circular(8),
              ),
            ),
            child: const Text(
              'Checkout',
              style: TextStyle(fontSize: 14, fontWeight: FontWeight.bold, color: Colors.white),
            ),
          ),
        ],
      ),
    );
  }
}

class ProductCard extends StatelessWidget {
  final Product product;
  final VoidCallback onTap;
  final VoidCallback onDoubleTap;
  final VoidCallback onLongPress;
  final VoidCallback onIncrement;
  final VoidCallback onDecrement;

  const ProductCard({
    super.key,
    required this.product,
    required this.onTap,
    required this.onDoubleTap,
    required this.onLongPress,
    required this.onIncrement,
    required this.onDecrement,
  });

  @override
  Widget build(BuildContext context) {
    String formattedPrice = product.price.toString().replaceAllMapped(
      RegExp(r'(\d{1,3})(?=(\d{3})+(?!\d))'),
      (Match m) => '${m[1]}.',
    );

    return GestureDetector(
      onTap: onTap,
      onDoubleTap: onDoubleTap,
      onLongPress: onLongPress,
      child: AnimatedContainer(
        duration: const Duration(milliseconds: 200),
        margin: const EdgeInsets.only(bottom: 12),
        padding: const EdgeInsets.all(12),
        decoration: BoxDecoration(
          color: Colors.white,
          borderRadius: BorderRadius.circular(12),
          border: Border.all(
            color: product.isSelected ? const Color(0xFF0D6EFD) : Colors.transparent,
            width: 2,
          ),
          boxShadow: [
            BoxShadow(
              color: Colors.black.withOpacity(0.04),
              blurRadius: 8,
              offset: const Offset(0, 2),
            ),
          ],
        ),
        child: Row(
          children: [
            ClipRRect(
              borderRadius: BorderRadius.circular(8),
              child: Image.network(
                product.imageUrl,
                width: 70,
                height: 70,
                fit: BoxFit.cover,
                errorBuilder: (context, error, stackTrace) => Container(
                  width: 70,
                  height: 70,
                  color: Colors.grey.shade300,
                  child: const Icon(Icons.image_not_supported),
                ),
              ),
            ),
            const SizedBox(width: 12),
            Expanded(
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    product.name,
                    style: const TextStyle(fontWeight: FontWeight.bold, fontSize: 13),
                  ),
                  Text(
                    product.brand,
                    style: const TextStyle(color: Colors.grey, fontSize: 11),
                  ),
                  const SizedBox(height: 4),
                  Text(
                    'Rp $formattedPrice',
                    style: const TextStyle(
                      color: Color(0xFF0D6EFD),
                      fontWeight: FontWeight.bold,
                      fontSize: 13,
                    ),
                  ),
                  const SizedBox(height: 6),
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      Row(
                        children: [
                          const Icon(Icons.favorite_border, size: 16, color: Colors.grey),
                          const SizedBox(width: 4),
                          Text(
                            '${product.likes}',
                            style: const TextStyle(fontSize: 12, color: Colors.grey),
                          ),
                        ],
                      ),
                      Row(
                        children: [
                          _buildQtyBtn(icon: Icons.remove, onPressed: onDecrement),
                          Padding(
                            padding: const EdgeInsets.symmetric(horizontal: 10),
                            child: Text(
                              '${product.quantity}',
                              style: const TextStyle(fontWeight: FontWeight.bold, fontSize: 13),
                            ),
                          ),
                          _buildQtyBtn(icon: Icons.add, onPressed: onIncrement, isPrimary: true),
                        ],
                      ),
                    ],
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildQtyBtn({required IconData icon, required VoidCallback onPressed, bool isPrimary = false}) {
    return InkWell(
      onTap: onPressed,
      borderRadius: BorderRadius.circular(4),
      child: Container(
        padding: const EdgeInsets.all(3),
        decoration: BoxDecoration(
          color: isPrimary ? const Color(0xFF0D6EFD) : Colors.grey.shade200,
          borderRadius: BorderRadius.circular(4),
        ),
        child: Icon(
          icon,
          size: 14,
          color: isPrimary ? Colors.white : Colors.black87,
        ),
      ),
    );
  }
}
