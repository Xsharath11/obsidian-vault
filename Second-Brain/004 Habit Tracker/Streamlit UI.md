Implemented a simple streamlit UI to signup, login and logout
This doc will focus on the issues faced while setting up the UI.

[[Additional Streamlit notes]]
### Issues faced:
1. Logout button persisting after clicking on it. Only disappears after second click
   ![[Pasted image 20250623172243.png]]
	^ How we are handling logout
	Why does this happen? 
	- Streamlit runs top to bottom on every interaction. (such as clicking a button)
	- `st.session_state["token"] = None` is updated, but this doesn't _immediately affect_ the current run.
	So on the **first run after clicking**:
	- `st.session_state["token"]` is still set when the top of the script runs.
	- Hence, `if st.session_state.get("token"):` still evaluates to `True`, and the logout button still appears.
	- Only on the **next rerun**, the updated `session_state` (with token `None`) takes effect.
	Solution:
	- Force a rerun by adding `st.rerun()` after `st.sucess()`
