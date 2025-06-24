#### Logout by unsetting token on streamlit vs unauth the token on the backend via an API:

❓ Why does unsetting the token in session state seem to work for logout?
Because your Streamlit frontend only checks whether st.session_state["token"] exists. When you remove it:

`st.session_state["token"] = None`

➡️ The frontend no longer thinks the user is authenticated, so it hides the logout button and shows the login/signup UI.
No API call is strictly needed for this frontend effect.
❓ So why have a /logout/ endpoint in the backend at all?

Because logout is not only a frontend concern. Backend logout is useful for:
🔐 1. Security:

If you're using token-based authentication (like DRF's TokenAuthentication), the logout endpoint can:

- Invalidate the token (i.e., delete it from the DB)
- Prevent reuse of the token even if it was leaked or stolen

❓ Then why doesn’t the logout button disappear instantly after using /logout/?
Because:
- Hitting /logout/ on the backend doesn’t modify Streamlit’s st.session_state automatically.
- You’re only updating the session after the request:

```
if st.sidebar.button("Logout"):
    response = requests.post(...)
    st.session_state["token"] = None
```

But Streamlit does not rerun the UI logic after this, unless you call `st.rerun()`.