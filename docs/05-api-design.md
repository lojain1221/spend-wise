# 05 — API design

The browser accesses the services through the API Gateway.

| Method | Path | Routed to | Purpose |
| --- | --- | --- | --- |
| POST | `/auth/register` | User | Create account |
| POST | `/auth/login` | User | Obtain access token |
| GET | `/users/me` | User | Current profile |
| GET | `/transactions` | Transaction | List transactions |
| POST | `/transactions` | Transaction | Add income or expense |
| DELETE | `/transactions/{id}` | Transaction | Delete transaction |
| GET | `/categories` | Transaction | List categories |
| POST | `/categories` | Transaction | Create category |
| GET | `/budgets` | Budget | List monthly budgets |
| POST | `/budgets` | Budget | Set a monthly budget |
| GET | `/notifications` | Notification | List alerts |
| PATCH | `/notifications/{id}/read` | Notification | Mark alert read |
| WS | `/ws/notifications` | Notification via Gateway | Push alerts to signed-in browser |

## Example request
`POST /transactions`
```json
{
  "category_id": "example-category-uuid",
  "type": "expense",
  "amount_cents": 3000,
  "currency": "EUR",
  "occurred_at": "2026-10-22T12:00:00Z",
  "note": "Groceries"
}
```

## Security
- Passwords are hashed, not stored in plain text.
- The User Service issues signed access tokens; the Gateway and downstream services validate authorization appropriately.
- User IDs come from authenticated identity, not untrusted request body fields.
- The WebSocket connection must authenticate before receiving personal notifications.

