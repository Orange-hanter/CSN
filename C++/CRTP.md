https://en.wikibooks.org/wiki/More_C%2B%2B_Idioms/Curiously_Recurring_Template_Pattern
# Curiously Recurring Template Pattern

Specialize a base class using the derived class as a template argument

In CRTP idiom, a class T inherits from a template that specializes on T.

```c++
class T : public X<T> {…};
```

This is valid only if the size of X<T> can be determined independently of T. Typically, the base class template will take advantage of the fact that member function bodies (definitions) are not instantiated until long after their declarations, and will use members of the derived class within its own member functions, via the use of a `static_cast`, e.g.:

```c++
  template <class Derived>
  struct base
  {
      void interface()
      {
          // ...
          static_cast<Derived*>(this)->implementation();
          // ...
      }
  
      static void static_interface()
      {
          // ...
          Derived::static_implementation();
          // ...
      }

      // The default implementation may be (if exists) or should be (otherwise) 
      // overridden by inheriting in derived classes (see below)
      void implementation();
      static void static_implementation();
  };

  // The Curiously Recurring Template Pattern (CRTP)
  struct derived_1 : base<derived_1>
  {
      // This class uses base variant of implementation
      //void implementation();
      
      // ... and overrides static_implementation
      static void static_implementation();
  };

  struct derived_2 : base<derived_2>
  {
      // This class overrides implementation
      void implementation();

      // ... and uses base variant of static_implementation
      //static void static_implementation();
  };
```