
## Basket Authorization Test

### Authentication baseline
- Test user basket ID: 6
- Authentication: SUCCESS
- Authentication mechanism: Bearer JWT
- Authenticated request to `/rest/basket/6`: HTTP 200
- JWT intentionally not recorded.

### Authorization test
- Authenticated user's basket: 6
- Requested different basket: 5
- Endpoint: GET /rest/basket/5
- Result: HTTP 200

### Finding
An authenticated user was able to retrieve a different basket by changing the basket object identifier.

Classification: Broken Object Level Authorization (BOLA / IDOR)

Expected control:
The server should verify that the authenticated user is authorized to access the requested basket object.

### Evidence
- evidence/basket-6.json — authenticated user's basket response
- evidence/basket-5.json — different basket response obtained using the same JWT


### Evidence verification
- basket-6.json returned basket ID: 6
- basket-5.json returned basket ID: 5
- Both responses were retrieved using the same authenticated JWT.
- This confirms access to a distinct basket object outside the authenticated user's basket.

