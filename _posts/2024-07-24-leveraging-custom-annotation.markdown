---
layout: post
title:  "Leveraging Custom Annotations in Kotlin"
date:   2024-06-13 10:00:00 GMT+2
categories: kotlin android annotation
---
![starting-image](/assets/images/post/annotation_img.webp){: width="512" height="512" }

In the world of software development, writing clean, readable, and maintainable code is a key priority. Kotlin, with its modern features and expressive syntax, provides many tools to help developers achieve this goal. One such powerful tool is annotations. Annotations in Kotlin can add extra information to your code, enforce constraints, or simplify repetitive tasks. In this article, I’ll explore custom annotations and how they can be used.


## What Are Annotations?

Annotations are like little notes you can attach to your code. These notes can tell the compiler or runtime to do something special with the annotated code. Kotlin comes with many built-in annotations like @Deprecated to mark old code, or @JvmStatic to indicate a static method. But you can also create your own custom annotations to fit your specific needs.

## Why Use Custom Annotations?

Creating your own annotations can make your code cleaner and more flexible. They help you separate concerns, enforce rules, and avoid code duplication. Let’s dive into a practical example to see how this works.

## Enforcing minimum payment amount using Annotation

Imagine you’re building a payment processing system that supports different payment methods and currencies. Each currency has a minimum amount that must be met for a transaction to be valid. We can use a custom annotation to enforce this rule.

### Step 1: Define Enums for Payment and Currency Types

```kotlin
enum class PaymentType(val type: String) {
    PAYPAL("paypal"), MASTER("master")
}

enum class CurrencyType(val minAmount: Double) {
    EURO(5.0), DOLLAR(4.0)
}
```

### Step 2: Create the Custom Annotation

Next, we create a custom annotation called MinAmountLimit. This annotation will mark parameters that need to be validated against a minimum amount constraint.
```kotlin
@Target(AnnotationTarget.VALUE_PARAMETER)
@Retention(AnnotationRetention.RUNTIME)
annotation class MinAmountLimit
```

### Step 3: Implement the Payment Class
```kotlin
import kotlin.reflect.full.declaredFunctions
import kotlin.reflect.full.findAnnotation
import kotlin.reflect.jvm.isAccessible

class Payment {

    fun pay(paymentType: PaymentType, currencyType: CurrencyType, amount: Double) {
        val func = Payment::class.declaredFunctions.find { it.name == paymentType.type }
        validateAmount(func, amount, currencyType)
        func?.call(this, currencyType, amount)
    }

    private fun validateAmount(func: KFunction<*>?, amount: Double, currencyType: CurrencyType) {
        func?.parameters?.forEach { parameter ->
            parameter.findAnnotation<MinAmountLimit>()?.let {
                if (amount < currencyType.minAmount) {
                    throw IllegalArgumentException("Amount should not be less than ${currencyType.minAmount} ${currencyType.name}")
                }
            }
        }
    }

    fun paypal(currencyType: CurrencyType, amount: Double) {
        println("Processed PayPal payment of $amount ${currencyType.name}")
    }

    fun master(currencyType: CurrencyType, @MinAmountLimit amount: Double) {
        println("Processed Master payment of $amount ${currencyType.name}")
    }
}
```
### How it works?
The pay method uses reflection to find the appropriate payment method based on the provided PaymentType and the validateAmount method checks if the amount is less than the minimum amount specified by the CurrencyType and throws an exception if it is. Otherwise, the amount is valid, the payment method is called using reflection.

# What are the advantages of using this approach?
Using annotations improves readability by making it easy to see at a glance what constraints are in place. They enhance code reusability, as the same annotation can be applied across different methods and parameters, reducing duplication. This approach also supports the separation of concerns by separating validation logic from business logic, resulting in cleaner and more maintainable code. Furthermore, custom annotations offer flexibility, allowing you to add behavior to your code without altering its core logic.

# Conclusion
Using custom annotations in Kotlin can significantly enhance your code by making it cleaner, more flexible, and easier to maintain. In the payment processing example, a custom annotation is used to enforce minimum payment amounts, demonstrating how annotations can help separate concerns and improve code quality.
