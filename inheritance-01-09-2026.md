# Inheritance in C++
## Lecture Notes — 01-09-2026

---

## 1. What is Inheritance?

Inheritance is a mechanism by which a new class (the **derived class** / **child class**) acquires the members (fields and methods) of an existing class (the **base class** / **parent class**), and can add or override behaviour on top of it.

Syntax:

```cpp
class Derived : public Base {
    // additional / overridden members
};
```

From `shapes.cpp`:

```cpp
class Rectangle : public Shape {
    ...
};
```

Here `Rectangle` is derived from `Shape` using **public inheritance** — everything `Shape` exposes publicly is also accessible through a `Rectangle` object.

---

## 2. As Many Classes, As Many Instances, As You Like

A program is not limited to one class or one object. You can define **any number of classes**, arrange them into hierarchies, and then create **any number of instances (objects)** of each class — completely independently of one another.

In `shapes.cpp` there are four classes: `Shape`, `Rectangle`, `Square`, `Circle`. `main()` then creates one object of each:

```cpp
Rectangle r1(10, 5);
Circle    c1(10);
Square    s1(10);
```

Nothing stops us from creating more, e.g. `Rectangle r2(3, 4), r3(7, 7);` — each object has its own independent copy of the member variables (`length`, `breadth`, `radius`, etc.), even though they all share the same class definition (and, if inherited, the same base-class definition too).

---

## 3. The Base Class: `Shape`

```cpp
class Shape {
public:
    float area() { return 0; }
    float circumference() { return 0; }
};
```

`Shape` acts as a common ancestor for every geometric shape in the program. It defines a generic interface — `area()` and `circumference()` — with placeholder (default) implementations that simply return `0`. Every shape "is-a" `Shape`, so any class that derives from it is guaranteed to provide these two methods, even if it doesn't (yet) give them meaningful logic.

---

## 4. Direct Children of `Shape`: `Rectangle` and `Circle`

Both `Rectangle` and `Circle` inherit directly from `Shape`. They are **siblings** — neither is related to the other except through their common parent.

### 4.1 `Rectangle`

```cpp
class Rectangle : public Shape {
private:
    const float length;
    const float breadth;

public:
    Rectangle(float l, float b) : length(l), breadth(b) {}

    float area() {
        cout << "Rectangle area ";
        return length * breadth;
    }

    float circumference() {
        cout << "Rectangle circumference ";
        return 2.0 * (length + breadth);
    }
};
```

- `length` and `breadth` are `private` and `const` — they can only be set once, via the constructor's **member initialiser list** (`: length(l), breadth(b)`), and never modified afterwards.
- `Rectangle` redefines `area()` and `circumference()` with real logic. Because these methods have the exact same name and signature as the ones in `Shape`, this is **method overriding**: a `Rectangle` object uses `Rectangle`'s versions instead of `Shape`'s placeholder versions.

### 4.2 `Circle`

```cpp
class Circle : public Shape {
private:
    static constexpr float PI = 3.141;
    const float radius;

public:
    Circle(float r) : radius(r) {}

    float area() { return PI * radius * radius; }
    float circumference() { return 2.0 * PI * radius; }
};
```

- `PI` is declared `static constexpr` — it belongs to the `Circle` class itself (not to any one object) and is a compile-time constant shared by every `Circle` instance.
- Like `Rectangle`, `Circle` overrides `area()` and `circumference()` from `Shape` with its own formulas.

---

## 5. Multi-level Inheritance: `Square` as a Child of `Rectangle`

```cpp
class Square : public Rectangle {
public:
    Square(float l) : Rectangle(l, l) {}
};
```

`Square` does **not** inherit directly from `Shape`. Instead it inherits from `Rectangle`, which itself inherits from `Shape`. This makes `Square` a **grandchild** of `Shape` and gives us a three-level chain:

```
Shape
 └── Rectangle
      └── Square
```

- The `Square` constructor takes a single side length `l` and delegates to the `Rectangle` constructor with `Rectangle(l, l)` — a square is just a rectangle whose length and breadth happen to be equal.
- Notice the commented-out code inside `Square`:

```cpp
/*
    float area() {
        cout << "Square area ";
        return length * length;
    }
    ...
*/
```

  This shows what a *separate* overriding implementation would have looked like, but it is disabled. Because `Square` does not define its own `area()` / `circumference()`, it simply **inherits** whatever version is visible in its nearest ancestor — which is `Rectangle`'s overridden version (not `Shape`'s original placeholder).

### A note on the terminology

Strictly speaking:

- `Rectangle` **overrides** `Shape::area()` and `Shape::circumference()` — it supplies its own implementation for a method that already existed in the parent.
- `Square` does **not** override anything itself — it simply **inherits** the already-overridden `Rectangle::area()` and `Rectangle::circumference()`, since it never redefines them.

So when `s1.area()` is called, what actually executes is `Rectangle::area()`, reached through inheritance — a useful illustration of how overriding and inheritance interact across multiple levels of a hierarchy.

---

## 6. Putting It Together: `main()`

```cpp
int main() {
    Rectangle r1(10, 5);
    Circle c1(10);
    Square s1(10);

    cout << r1.area() << " " << r1.circumference() << endl;
    cout << c1.area() << " " << c1.circumference() << endl;
    cout << s1.area() << " " << s1.circumference() << endl;

    return 0;
}
```

- `r1`, `c1`, and `s1` are three completely independent objects of three different classes.
- `r1.area()` and `r1.circumference()` call `Rectangle`'s own overridden versions.
- `c1.area()` and `c1.circumference()` call `Circle`'s own overridden versions.
- `s1.area()` and `s1.circumference()` resolve to `Rectangle`'s versions (via inheritance, as explained in Section 5), since `Square` provides none of its own — printing `"Rectangle area "` and `"Rectangle circumference "` even though the object being used is a `Square`.

---

## 7. Summary

| Class       | Parent      | Defines own `area()`/`circumference()`? | Where the method actually comes from |
|-------------|-------------|------------------------------------------|----------------------------------------|
| `Shape`     | —           | Yes (placeholder, returns 0)             | `Shape`                                |
| `Rectangle` | `Shape`     | Yes (overrides `Shape`)                  | `Rectangle`                            |
| `Circle`    | `Shape`     | Yes (overrides `Shape`)                  | `Circle`                               |
| `Square`    | `Rectangle` | No                                        | `Rectangle` (inherited, not overridden)|

Key takeaways:

1. A program can have as many classes, in as many parent–child relationships, and as many instances of each, as needed.
2. Public inheritance (`class Derived : public Base`) lets a derived class reuse and extend a base class's members.
3. Inheritance can be **multi-level** (`Shape → Rectangle → Square`), not just a single parent–child pair.
4. **Overriding** means a derived class supplies its own implementation of a method with the same signature as its base class.
5. If a derived class does not override an inherited method, calls to that method fall back to the nearest ancestor's implementation — which may itself be an overridden version, not the original one.
