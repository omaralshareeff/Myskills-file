# Flutter examples

Shapes to apply. Match the project's types, state library, and names. Do not copy a Riverpod snippet into a Bloc codebase, or the reverse.

## Contents

- Policy leaves the widget
- One state value
- Extract on the third copy
- Parse at the boundary
- A flag that is really two widgets
- A test names the behavior

## Policy leaves the widget

The widget paints a decision it did not make. The clock is an argument, so the rule can be tested without pumping a widget.

```dart
const refundWindow = Duration(days: 14);

bool eligibleForRefund(Order order, DateTime now) {
  final deliveredAt = order.deliveredAt;
  if (order.status != OrderStatus.delivered || deliveredAt == null) {
    return false;
  }
  if (order.refunded) {
    return false;
  }
  return now.difference(deliveredAt) < refundWindow;
}

class OrderTile extends StatelessWidget {
  const OrderTile({super.key, required this.order, required this.canRefund});

  final Order order;
  final bool canRefund;

  @override
  Widget build(BuildContext context) {
    return ListTile(
      title: Text(order.number),
      trailing: canRefund ? const Text('Refund') : null,
    );
  }
}
```

The rejected shape puts `DateTime.now()`, `deliveredAt!`, and the 14-day rule inside `build`. The next change to the window then requires a widget test, and the bang hides a missing timestamp.

## One state value

Parallel flags can be loading and failed at once. A sealed type cannot.

```dart
sealed class InvoiceViewState {}

final class InvoiceLoading extends InvoiceViewState {}

final class InvoiceReady extends InvoiceViewState {
  InvoiceReady(this.invoice);

  final Invoice invoice;
}

final class InvoiceFailed extends InvoiceViewState {
  InvoiceFailed(this.reason);

  final String reason;
}
```

Switch on `InvoiceViewState` with an arm for each type. Add the new arm when a new type appears.

## Extract on the third copy

Two call sites that format an amount as `'${amount.toStringAsFixed(2)} SAR'` may stay duplicated when one is a receipt line and the other is an input hint. They will change for different reasons.

The third copy, or the second copy of a real rule such as refund eligibility, becomes a named function. It does not become `AppUtils.check`.

## Parse at the boundary

A repository returns `Customer`. The casts and the missing-field failures live in `Customer.fromJson`. Callers never receive `Map<String, dynamic>`, and a widget never reads `json['name'] as String`.

## A flag that is really two widgets

`SummaryCard({required bool compact})` with two layouts inside is two widgets: `SummaryCard` and `CompactSummaryCard`. Share a private child only when both paint the same element for the same reason.

Two screens that both show a title are not a `GenericCard(mode: ...)`. Build the two widgets. Extract when a third screen needs the same layout rule, and name that rule.

## A test names the behavior

```dart
test('eligibleForRefund is false when the window has elapsed', () {
  final order = deliveredOrder(at: DateTime.utc(2026, 1, 1));

  expect(
    eligibleForRefund(order, DateTime.utc(2026, 2, 1)),
    isFalse,
  );
});
```

The test constructs the clock. It does not call `DateTime.now()`, and it does not assert on private widget text that the user never reads.
