# Reto_04_POO
### Juan Pablo Rodríguez Cruz

## 1. **Create a repo with the class exercise**

```python
from math import sqrt, acos, degrees

class Point:
    """Clase que representa un punto en un plano bidimensional."""
    def __init__(self, x: float = 0, y: float = 0):
        self._x = x
        self._y = y

    def get_x(self):
        return self._x

    def set_x(self, value):
        self._x = value

    def get_y(self):
        return self._y

    def set_y(self, value):
        self._y = value

    def compute_distance(self, other_point):
        return sqrt((self.get_x() - other_point.get_x())**2 + (self.get_y() - other_point.get_y())**2)


class Line:
    """Clase que representa una línea en un plano bidimensional."""
    def __init__(self, start_point: Point, end_point: Point):
        self._start_point = start_point
        self._end_point = end_point
        self._length = self.compute_length()

    def get_start_point(self):
        return self._start_point

    def set_start_point(self, value):
        self._start_point = value
        self._length = self.compute_length()

    def get_end_point(self):
        return self._end_point

    def set_end_point(self, value):
        self._end_point = value
        self._length = self.compute_length()

    def get_length(self):
        return self._length

    def compute_length(self):
        return self.get_start_point().compute_distance(self.get_end_point())


class Shape:
    """Superclase que representa una figura geométrica."""
    def __init__(self, vertices: list[Point], edges: list[Line]):
        self.vertices = vertices
        self.edges = edges
        self.inner_angles = self.compute_inner_angles()
        self.is_regular = self.check_regular()

    def compute_area(self):
        raise NotImplementedError("Este método debe ser implementado por las subclases.")

    def compute_perimeter(self):
        return sum(edge.get_length() for edge in self.edges)

    def compute_inner_angles(self):
        """Calcula los ángulos internos (solo para polígonos convexos)."""
        angles = []
        for i in range(len(self.vertices)):
            p1 = self.vertices[i - 1]
            p2 = self.vertices[i]
            p3 = self.vertices[(i + 1) % len(self.vertices)]

            a = p1.compute_distance(p2)
            b = p2.compute_distance(p3)
            c = p1.compute_distance(p3)

            angle = acos((a**2 + b**2 - c**2) / (2 * a * b))
            angles.append(degrees(angle))
        return angles

    def check_regular(self):
        """Verifica si la figura es regular."""
        if len(self.edges) < 3:
            return False

        first_length = self.edges[0].get_length()
        first_angle = self.inner_angles[0]

        return all(edge.get_length() == first_length for edge in self.edges) and \
               all(angle == first_angle for angle in self.inner_angles)


class Rectangle(Shape):
    """Clase que representa un rectángulo."""
    def __init__(self, bottom_left: Point, top_right: Point):
        width = top_right.get_x() - bottom_left.get_x()
        height = top_right.get_y() - bottom_left.get_y()

        vertices = [
            bottom_left,
            Point(top_right.get_x(), bottom_left.get_y()),
            top_right,
            Point(bottom_left.get_x(), top_right.get_y())
        ]

        edges = [
            Line(vertices[0], vertices[1]),
            Line(vertices[1], vertices[2]),
            Line(vertices[2], vertices[3]),
            Line(vertices[3], vertices[0])
        ]

        super().__init__(vertices, edges)
        self.width = width
        self.height = height

    def compute_area(self):
        return self.width * self.height


class Square(Rectangle):
    """Clase que representa un cuadrado (subclase de Rectángulo)."""
    def __init__(self, bottom_left: Point, side_length: float):
        top_right = Point(bottom_left.get_x() + side_length, bottom_left.get_y() + side_length)
        super().__init__(bottom_left, top_right)

    def compute_area(self):
        return self.width**2


class Triangle(Shape):
    """Clase que representa un triángulo."""
    def __init__(self, vertex1: Point, vertex2: Point, vertex3: Point):
        vertices = [vertex1, vertex2, vertex3]

        edges = [
            Line(vertices[0], vertices[1]),
            Line(vertices[1], vertices[2]),
            Line(vertices[2], vertices[0])
        ]

        super().__init__(vertices, edges)

    def compute_area(self):
        """Calcula el área usando la fórmula de Herón."""
        a, b, c = (edge.get_length() for edge in self.edges)
        s = (a + b + c) / 2
        return (s * (s - a) * (s - b) * (s - c)) ** 0.5


class Isosceles(Triangle):
    """Clase que representa un triángulo isósceles."""
    def __init__(self, vertex1: Point, vertex2: Point, vertex3: Point):
        super().__init__(vertex1, vertex2, vertex3)
        a, b, c = (edge.get_length() for edge in self.edges)
        if not (a == b or b == c or a == c):
            raise ValueError("No es un triángulo isósceles.")


class Equilateral(Triangle):
    """Clase que representa un triángulo equilátero."""
    def __init__(self, vertex1: Point, vertex2: Point, vertex3: Point):
        super().__init__(vertex1, vertex2, vertex3)
        a, b, c = (edge.get_length() for edge in self.edges)
        if not (a == b == c):
            raise ValueError("No es un triángulo equilátero.")


class Scalene(Triangle):
    """Clase que representa un triángulo escaleno."""
    def __init__(self, vertex1: Point, vertex2: Point, vertex3: Point):
        super().__init__(vertex1, vertex2, vertex3)
        a, b, c = (edge.get_length() for edge in self.edges)
        if a == b or b == c or a == c:
            raise ValueError("No es un triángulo escaleno.")


class TriRectangle(Triangle):
    """Clase que representa un triángulo rectángulo."""
    def __init__(self, vertex1: Point, vertex2: Point, vertex3: Point):
        super().__init__(vertex1, vertex2, vertex3)
        a, b, c = sorted(edge.get_length() for edge in self.edges)
        if not abs(c**2 - (a**2 + b**2)) < 1e-9:
            raise ValueError("No es un triángulo rectángulo.")


# Ejemplo de uso
if __name__ == "__main__":
    # Crear puntos
    p1 = Point(0, 0)
    p2 = Point(4, 0)
    p3 = Point(2, 3)

    # Crear un triángulo equilátero
    try:
        eq_triangle = Equilateral(Point(0, 0), Point(3, 0), Point(1.5, sqrt(3) * 1.5))
        print(f"Área del triángulo equilátero: {eq_triangle.compute_area()}")
        print(f"Perímetro del triángulo equilátero: {eq_triangle.compute_perimeter()}")
    except ValueError as e:
        print(e)

    # Crear un triángulo rectángulo
    try:
        tri_rectangle = TriRectangle(p1, p2, p3)
        print(f"Área del triángulo rectángulo: {tri_rectangle.compute_area()}")
        print(f"Perímetro del triángulo rectángulo: {tri_rectangle.compute_perimeter()}")
    except ValueError as e:
        print(e)

    # Crear un rectángulo
    rect = Rectangle(p1, Point(4, 4))
    print(f"Área del rectángulo: {rect.compute_area()}")
    print(f"Perímetro del rectángulo: {rect.compute_perimeter()}")

    # Crear un cuadrado
    square = Square(p1, 4)
    print(f"Área del cuadrado: {square.compute_area()}")
    print(f"Perímetro del cuadrado: {square.compute_perimeter()}")

```
## 2. The restaurant revisted
- Add setters and getters to all subclasses for menu item
- Override calculate_total_price() according to the order composition (e.g if the order includes a main course apply some disccount on beverages)
- Add the class Payment() following the class example.

``` python
import os

class MenuItem:
    """Base class for a menu item."""

    def __init__(self, name, price):
        self._name = name
        self._price = price

    def get_name(self):
        return self._name

    def set_name(self, name):
        self._name = name

    def get_price(self):
        return self._price

    def set_price(self, price):
        self._price = price

    def calculate_price(self):
        return self._price


class Beverage(MenuItem):
    """Subclass for beverages."""

    def __init__(self, name, price, is_alcoholic):
        super().__init__(name, price)
        self._is_alcoholic = is_alcoholic

    def get_is_alcoholic(self):
        return self._is_alcoholic

    def set_is_alcoholic(self, value):
        self._is_alcoholic = value


class Appetizer(MenuItem):
    """Subclass for appetizers."""

    pass


class MainCourse(MenuItem):
    """Subclass for main courses."""

    def __init__(self, name, price, is_vegetarian):
        super().__init__(name, price)
        self._is_vegetarian = is_vegetarian

    def get_is_vegetarian(self):
        return self._is_vegetarian

    def set_is_vegetarian(self, value):
        self._is_vegetarian = value


class Order:
    """Class representing a customer's order."""

    def __init__(self):
        self.items = []

    def add_item(self, item):
        if isinstance(item, MenuItem):
            self.items.append(item)
        else:
            raise ValueError("The item must be a MenuItem object.")

    def calculate_total(self):
        return sum(item.calculate_price() for item in self.items)

    def apply_discount(self, discount_percentage):
        total = self.calculate_total()
        discount = total * (discount_percentage / 100)
        return total - discount

    def calculate_total_price(self):
        total = self.calculate_total()

        # Override for main course and beverage discount
        has_main_course = any(isinstance(item, MainCourse) for item in self.items)
        has_beverage = any(isinstance(item, Beverage) for item in self.items)

        if has_main_course and has_beverage:
            total *= 0.9  # 10% discount on beverages if main course exists

        return total

    def show_order_summary(self):
        print("Resumen del pedido:")
        for item in self.items:
            print(f"{item.get_name()}: ${item.get_price():.2f}")
        print(f"Total: ${self.calculate_total_price():.2f}")

    @staticmethod
    def limpiar_pantalla():
        """Limpia la consola."""
        os.system("cls" if os.name == "nt" else "clear")

    @staticmethod
    def continuar():
        """Pausa la ejecución hasta que el usuario presione Enter."""
        input("\nPresiona Enter para continuar...")
        Order.limpiar_pantalla()


class Payment:
    """Abstract base class for payment methods."""

    def pay(self, amount):
        raise NotImplementedError("Subclasses must implement the pay method.")


class CreditCard(Payment):
    def __init__(self, card_number, cvv):
        self.card_number = card_number
        self.cvv = cvv

    def pay(self, amount):
        print(f"Pagando ${amount:.2f} con tarjeta terminada en {self.card_number[-4:]}")


class Cash(Payment):
    def __init__(self, amount_given):
        self.amount_given = amount_given

    def pay(self, amount):
        if self.amount_given >= amount:
            change = self.amount_given - amount
            print(f"Pago realizado en efectivo. Cambio: ${change:.2f}")
        else:
            shortage = amount - self.amount_given
            print(f"Fondos insuficientes. Faltan ${shortage:.2f} para completar el pago.")


def main_menu():
    menu = [
        Beverage("Gaseosa 600 mL", 7000, False),
        Beverage("Cerveza artesanal", 15000, True),
        Beverage("Cerveza Heineken botella", 10000, True),
        Beverage("Limonada de coco", 10000, False),
        Beverage("Agua", 5000, False),
        Appetizer("Empanadas de pollo(4und)", 10000),
        Appetizer("Empanadas de carne(4und)", 10000),
        Appetizer("Empanadas de camarón(4und)", 12000),
        Appetizer("Chunchullo", 18000),
        Appetizer("Ceviche", 15000),
        MainCourse("Churrasco", 40000, False),
        MainCourse("Medio Churrasco", 28000, False),
        MainCourse("Bandeja Paisa", 38000, False),
        MainCourse("Hamburguesa", 33000, False),
        MainCourse("Hamburguesa vegetariana", 36000, True),
        MainCourse("Mojarra", 38000, False)
    ]

    order = Order()

    while True:
        Order.limpiar_pantalla()
        print("Menú del Restaurante:")
        print("\n--- Bebidas ---")
        for i, item in enumerate(menu, start=1):
            if isinstance(item, Beverage):
                print(f"{i}. {item.get_name()} - ${item.get_price():.2f}")
        print("\n--- Aperitivos ---")
        for i, item in enumerate(menu, start=1):
            if isinstance(item, Appetizer):
                print(f"{i}. {item.get_name()} - ${item.get_price():.2f}")
        print("\n--- Platos principales ---")
        for i, item in enumerate(menu, start=1):
            if isinstance(item, MainCourse):
                print(f"{i}. {item.get_name()} - ${item.get_price():.2f}")

        try:
            choices = input("\nIngrese los números de los items separados por comas: ").strip()

            choices = [int(choice) - 1 for choice in choices.split(",") if choice.strip().isdigit()]
            for choice in choices:
                if 0 <= choice < len(menu):
                    order.add_item(menu[choice])
                    print(f"{menu[choice].get_name()} agregado al pedido.")
                else:
                    print(f"El número {choice + 1} no es válido y se ignoró.")
        except ValueError:
            print("\nEntrada inválida. Por favor, ingrese solo números separados por comas.")

        confirm = input("\n¿Desea agregar más items? (s/n): ").strip().lower()
        if confirm != 's':
            break

    Order.limpiar_pantalla()
    print("\nResumen Final del Pedido:")
    order.show_order_summary()

    confirm = input("\n¿Confirma su pedido? (s/n): ").strip().lower()
    if confirm == 's':
        payment_method = input("\nSeleccione método de pago (1: Tarjeta, 2: Efectivo): ").strip()
        if payment_method == '1':
            card_number = input("Ingrese el número de tarjeta: ").strip()
            cvv = input("Ingrese el CVV: ").strip()
            card = CreditCard(card_number, cvv)
            card.pay(order.calculate_total_price())
        elif payment_method == '2':
            try:
                amount_given = float(input("Ingrese el monto entregado: ").strip())
                cash = Cash(amount_given)
                cash.pay(order.calculate_total_price())
            except ValueError:
                print("\nMonto ingresado no válido.")
        else:
            print("\nMétodo de pago no válido")
    else:
        print("\nEl pedido fue cancelado.")


if __name__ == "__main__":
    main_menu()

```
