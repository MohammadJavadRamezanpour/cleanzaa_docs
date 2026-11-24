# Customer and worker space switcher

The **Switch space** control is at the top of the customer and worker screens, including sign-in, services, booking, order history, and the worker request inbox. Both destinations are visible together, and the current space is highlighted and announced by assistive technology.

**Customer** opens `/` and **Worker** opens `/worker`. The control is available before and after sign-in. It navigates to the selected space's home screen; each destination checks whether the browser has a session that can access that space and offers sign-in when needed. The control does not change account roles or sign users in or out on its own.

## Verification

On a narrow mobile viewport, open the customer home and use the header to switch to Worker and back. Verify that both links are visible before scrolling, have comfortable touch targets, and show a visible focus outline when reached with the keyboard. On `/services`, `/book`, and `/orders`, verify Customer is marked as current; on `/worker/requests`, verify Worker is marked as current. Run the frontend lint, format, type, test, contract, and production build checks.
