# Python examples

Shapes to apply. Match the project's framework, schema library, and test runner. A Django project keeps Django types. A script does not grow a package for one function.

## Contents

- Decide and save are different functions
- A flag that is really two functions
- Defaults, failures, and the edge
- Import time stays pure
- Extract on the third copy
- A test names the behavior

## Decide and save are different functions

Parsing, the approval rule, and the database are three reasons to change. `main` composes them. Each piece can be tested without the others.

```python
from dataclasses import dataclass
from decimal import Decimal, InvalidOperation
from typing import Mapping


APPROVAL_LIMIT = Decimal("10000")


class InvoicingError(Exception):
    pass


class InvoicePayloadError(InvoicingError):
    pass


@dataclass(frozen=True)
class Invoice:
    invoice_id: str
    total: Decimal
    approved: bool


def parse_invoice(payload: Mapping[str, object]) -> Invoice:
    invoice_id = payload.get("invoice_id")
    raw_total = payload.get("total")
    if not isinstance(invoice_id, str) or invoice_id == "":
        raise InvoicePayloadError("invoice_id is missing")
    if not isinstance(raw_total, str):
        raise InvoicePayloadError("total is missing or not a decimal string")
    try:
        total = Decimal(raw_total)
    except InvalidOperation as err:
        raise InvoicePayloadError(f"total is not a decimal: {raw_total}") from err
    return Invoice(
        invoice_id=invoice_id,
        total=total,
        approved=payload.get("approved") is True,
    )


def approval_required(invoice: Invoice) -> bool:
    return invoice.total > APPROVAL_LIMIT and not invoice.approved
```

The rejected shape is one `process_invoice(path)` that reads a file, prints `"bad"`, and calls `db.save` on a `dict`. The next change to the limit requires a database, and a missing total becomes a `KeyError` from the middle of the function.

`InvoicePayloadError` is a subclass of the package's base exception. The store lives in `store.py` and accepts an `Invoice`, not a `dict`.

## A flag that is really two functions

`export(rows, as_csv: bool = False)` is `export_csv` and `export_json`. Share a private `_as_records` only when both exports prepare the same records for the same reason.

Two callers that both need a title and a subtitle are not a `GenericCard(mode=...)`. Write the two functions. Extract when a third caller shares the rule, and name that rule.

## Defaults, failures, and the edge

```python
def collect(fetched: Item, items: list[Item] | None = None) -> list[Item]:
    found = list(items or [])
    found.append(fetched)
    return found
```

The default list is created inside the function, so callers do not share one list. `fetch` is called by the composer, and its failure propagates as a specific exception. The rejected shape uses `items=[]` and `except Exception: return None`, which hides a failure inside an empty result and keeps state between calls.

## Import time stays pure

A module defines `build_client(settings: Settings) -> Client`. `main` reads the environment, builds `Settings`, and calls `build_client`. The module does not open a socket or call `os.environ` while it is imported. Tests can then import the rule without credentials.

## Extract on the third copy

Two places that format a decimal as money may stay duplicated when one is a receipt and the other is a form hint.

The third copy, or the second copy of a real rule such as `approval_required`, becomes a named function in the module that owns the rule. It does not become `utils.check`.

## A test names the behavior

```python
def test_approval_required_when_total_exceeds_limit_and_not_approved():
    invoice = Invoice(
        invoice_id="inv-1",
        total=Decimal("10001"),
        approved=False,
    )

    assert approval_required(invoice) is True
```

The test builds the value and calls the public function. It does not mock `approval_required`, and it does not need a database.
