---
title: The PaymentCardNumber class
---
This document covers the <SwmToken path="pydantic/v1/types.py" pos="1037:12:12" line-data="    def validate_length_for_brand(cls, card_number: &#39;PaymentCardNumber&#39;) -&gt; &#39;PaymentCardNumber&#39;:">`PaymentCardNumber`</SwmToken> class. We'll address:

1. What <SwmToken path="pydantic/v1/types.py" pos="1037:12:12" line-data="    def validate_length_for_brand(cls, card_number: &#39;PaymentCardNumber&#39;) -&gt; &#39;PaymentCardNumber&#39;:">`PaymentCardNumber`</SwmToken> is and its purpose
2. All variables and functions defined in <SwmToken path="pydantic/v1/types.py" pos="1037:12:12" line-data="    def validate_length_for_brand(cls, card_number: &#39;PaymentCardNumber&#39;) -&gt; &#39;PaymentCardNumber&#39;:">`PaymentCardNumber`</SwmToken>

# What is <SwmToken path="pydantic/v1/types.py" pos="1037:12:12" line-data="    def validate_length_for_brand(cls, card_number: &#39;PaymentCardNumber&#39;) -&gt; &#39;PaymentCardNumber&#39;:">`PaymentCardNumber`</SwmToken>

<SwmToken path="pydantic/v1/types.py" pos="1037:12:12" line-data="    def validate_length_for_brand(cls, card_number: &#39;PaymentCardNumber&#39;) -&gt; &#39;PaymentCardNumber&#39;:">`PaymentCardNumber`</SwmToken> is a class in pydantic that represents and validates payment card numbers, such as credit or debit card numbers. It is designed to ensure that card numbers conform to industry standards, including length, digit-only constraints, brand identification, and the Luhn check digit algorithm. This class is useful for securely handling and validating payment card data in applications.

<SwmSnippet path="/pydantic/v1/types.py" line="983">

---

The class variable <SwmToken path="pydantic/v1/types.py" pos="983:1:1" line-data="    strip_whitespace: ClassVar[bool] = True">`strip_whitespace`</SwmToken> is set to True, indicating that any whitespace in the card number should be removed during validation.

```python
    strip_whitespace: ClassVar[bool] = True
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/types.py" line="984">

---

The class variable <SwmToken path="pydantic/v1/types.py" pos="984:1:1" line-data="    min_length: ClassVar[int] = 12">`min_length`</SwmToken> defines the minimum allowed length for a payment card number, which is 12 digits.

```python
    min_length: ClassVar[int] = 12
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/types.py" line="985">

---

The class variable <SwmToken path="pydantic/v1/types.py" pos="985:1:1" line-data="    max_length: ClassVar[int] = 19">`max_length`</SwmToken> sets the maximum allowed length for a payment card number, which is 19 digits.

```python
    max_length: ClassVar[int] = 19
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/types.py" line="986">

---

The instance variable <SwmToken path="pydantic/v1/types.py" pos="986:1:1" line-data="    bin: str">`bin`</SwmToken> stores the first six digits of the card number, which typically identify the issuing bank.

```python
    bin: str
    last4: str
    brand: PaymentCardBrand

    def __init__(self, card_number: str):
        self.bin = card_number[:6]
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/types.py" line="987">

---

The instance variable <SwmToken path="pydantic/v1/types.py" pos="987:1:1" line-data="    last4: str">`last4`</SwmToken> holds the last four digits of the card number, commonly used for display or identification purposes.

```python
    last4: str
    brand: PaymentCardBrand

    def __init__(self, card_number: str):
        self.bin = card_number[:6]
        self.last4 = card_number[-4:]
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/types.py" line="988">

---

The instance variable <SwmToken path="pydantic/v1/types.py" pos="988:1:1" line-data="    brand: PaymentCardBrand">`brand`</SwmToken> identifies the card brand (such as Visa, Mastercard, or Amex) based on the card number.

```python
    brand: PaymentCardBrand

    def __init__(self, card_number: str):
        self.bin = card_number[:6]
        self.last4 = card_number[-4:]
        self.brand = self._get_brand(card_number)
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/types.py" line="990">

---

The <SwmToken path="pydantic/v1/types.py" pos="990:3:3" line-data="    def __init__(self, card_number: str):">`__init__`</SwmToken> function initializes a <SwmToken path="pydantic/v1/types.py" pos="1037:12:12" line-data="    def validate_length_for_brand(cls, card_number: &#39;PaymentCardNumber&#39;) -&gt; &#39;PaymentCardNumber&#39;:">`PaymentCardNumber`</SwmToken> instance by extracting the BIN, last four digits, and determining the card brand from the provided card number.

```python
    def __init__(self, card_number: str):
        self.bin = card_number[:6]
        self.last4 = card_number[-4:]
        self.brand = self._get_brand(card_number)
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/types.py" line="995">

---

The class method <SwmToken path="pydantic/v1/types.py" pos="996:3:3" line-data="    def __get_validators__(cls) -&gt; &#39;CallableGenerator&#39;:">`__get_validators__`</SwmToken> yields a sequence of validators that are applied to the card number, including string validation, whitespace stripping, length checks, digit validation, Luhn check, and brand-specific length validation.

```python
    @classmethod
    def __get_validators__(cls) -> 'CallableGenerator':
        yield str_validator
        yield constr_strip_whitespace
        yield constr_length_validator
        yield cls.validate_digits
        yield cls.validate_luhn_check_digit
        yield cls
        yield cls.validate_length_for_brand

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/types.py" line="1005">

---

The property <SwmToken path="pydantic/v1/types.py" pos="1006:3:3" line-data="    def masked(self) -&gt; str:">`masked`</SwmToken> returns a masked version of the card number, showing only the BIN and last four digits, with the intervening digits replaced by asterisks.

```python
    @property
    def masked(self) -> str:
        num_masked = len(self) - 10  # len(bin) + len(last4) == 10
        return f'{self.bin}{"*" * num_masked}{self.last4}'

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/types.py" line="1011">

---

The class method <SwmToken path="pydantic/v1/types.py" pos="1011:3:3" line-data="    def validate_digits(cls, card_number: str) -&gt; str:">`validate_digits`</SwmToken> ensures that the card number consists only of digits, raising an error if non-digit characters are present.

```python
    def validate_digits(cls, card_number: str) -> str:
        if not card_number.isdigit():
            raise errors.NotDigitError
        return card_number
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/types.py" line="1017">

---

The class method <SwmToken path="pydantic/v1/types.py" pos="1017:3:3" line-data="    def validate_luhn_check_digit(cls, card_number: str) -&gt; str:">`validate_luhn_check_digit`</SwmToken> validates the card number using the Luhn algorithm, which is a standard checksum used to verify card numbers.

```python
    def validate_luhn_check_digit(cls, card_number: str) -> str:
        """
        Based on: https://en.wikipedia.org/wiki/Luhn_algorithm
        """
        sum_ = int(card_number[-1])
        length = len(card_number)
        parity = length % 2
        for i in range(length - 1):
            digit = int(card_number[i])
            if i % 2 == parity:
                digit *= 2
            if digit > 9:
                digit -= 9
            sum_ += digit
        valid = sum_ % 10 == 0
        if not valid:
            raise errors.LuhnValidationError
        return card_number
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/types.py" line="1037">

---

The class method <SwmToken path="pydantic/v1/types.py" pos="1037:3:3" line-data="    def validate_length_for_brand(cls, card_number: &#39;PaymentCardNumber&#39;) -&gt; &#39;PaymentCardNumber&#39;:">`validate_length_for_brand`</SwmToken> checks that the card number length matches the expected length for its brand (e.g., 16 for Mastercard, 13/16/19 for Visa, 15 for Amex).

```python
    def validate_length_for_brand(cls, card_number: 'PaymentCardNumber') -> 'PaymentCardNumber':
        """
        Validate length based on BIN for major brands:
        https://en.wikipedia.org/wiki/Payment_card_number#Issuer_identification_number_(IIN)
        """
        required_length: Union[None, int, str] = None
        if card_number.brand in PaymentCardBrand.mastercard:
            required_length = 16
            valid = len(card_number) == required_length
        elif card_number.brand == PaymentCardBrand.visa:
            required_length = '13, 16 or 19'
            valid = len(card_number) in {13, 16, 19}
        elif card_number.brand == PaymentCardBrand.amex:
            required_length = 15
            valid = len(card_number) == required_length
        else:
            valid = True
        if not valid:
            raise errors.InvalidLengthForBrand(brand=card_number.brand, required_length=required_length)
        return card_number
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/types.py" line="1059">

---

The static method <SwmToken path="pydantic/v1/types.py" pos="1059:3:3" line-data="    def _get_brand(card_number: str) -&gt; PaymentCardBrand:">`_get_brand`</SwmToken> determines the card brand by inspecting the card number's prefix, distinguishing between Visa, Mastercard, Amex, and other brands.

```python
    def _get_brand(card_number: str) -> PaymentCardBrand:
        if card_number[0] == '4':
            brand = PaymentCardBrand.visa
        elif 51 <= int(card_number[:2]) <= 55:
            brand = PaymentCardBrand.mastercard
        elif card_number[:2] in {'34', '37'}:
            brand = PaymentCardBrand.amex
        else:
            brand = PaymentCardBrand.other
        return brand
```

---

</SwmSnippet>

# Usage

## <SwmToken path="pydantic/v1/types.py" pos="1037:12:12" line-data="    def validate_length_for_brand(cls, card_number: &#39;PaymentCardNumber&#39;) -&gt; &#39;PaymentCardNumber&#39;:">`PaymentCardNumber`</SwmToken>

<SwmToken path="pydantic/v1/types.py" pos="1037:12:12" line-data="    def validate_length_for_brand(cls, card_number: &#39;PaymentCardNumber&#39;) -&gt; &#39;PaymentCardNumber&#39;:">`PaymentCardNumber`</SwmToken> is used in validation logic that involves the Luhn algorithm, which is a checksum formula used to validate various identification numbers, including credit card numbers. In one usage, the class is involved in a function that attempts to append a digit to a card number and validate it using the Luhn check digit method, ensuring the card number is valid according to this algorithm.

## <SwmToken path="pydantic/v1/types.py" pos="1037:12:12" line-data="    def validate_length_for_brand(cls, card_number: &#39;PaymentCardNumber&#39;) -&gt; &#39;PaymentCardNumber&#39;:">`PaymentCardNumber`</SwmToken>

Additionally, <SwmToken path="pydantic/v1/types.py" pos="1037:12:12" line-data="    def validate_length_for_brand(cls, card_number: &#39;PaymentCardNumber&#39;) -&gt; &#39;PaymentCardNumber&#39;:">`PaymentCardNumber`</SwmToken> is registered with a testing strategy that uses regular expressions to generate card number patterns. This strategy includes a mapping function that adds a valid Luhn check digit to the generated card number strings, facilitating property-based testing of card number validation.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
