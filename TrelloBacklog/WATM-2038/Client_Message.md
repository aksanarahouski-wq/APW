# WATM-2038: Client Message Draft

**To:** Laura Perry / APW Team
**Re:** WATM-2038 — Additional Status (Remove "Offline" for SIM/Systech/M5/M5-302)

---

Hi Laura,

We've reviewed the request to remove the "Offline" option from the Additional Status field for SIM and Systech manufacturers and M5/M5-302 models.

After reviewing our current architecture, we have two approaches we'd like to present for your consideration:

**Option 1: Hardcoded Rule**
We add a rule in the code that removes "Offline" from the Additional Status dropdown specifically for the manufacturers and models you listed. This is quick to implement and addresses your exact request.

The tradeoff is that any future changes to which statuses are available for which models would require a code change and deployment on our side. If you need to add or remove status restrictions for other models down the road, you'd need to come to us each time.

**Option 2: Admin-Configurable Status per Model**
We extend the Device Model management page in the admin panel so you can control which Additional Statuses are available for each model. You'd simply go to the model's settings and check/uncheck which statuses should be available.

This takes a bit more upfront effort to build, but gives you full control going forward — no need to involve us if you want to restrict or enable statuses for other models in the future.

**Our Recommendation:** If there's a chance you'll want to manage status availability for other models or manufacturers in the future, Option 2 gives you that flexibility without needing to wait on us. If this is a one-time change and you don't anticipate needing to adjust this again, Option 1 gets it done quickly.

A few clarifying questions before we proceed:

1. Should this restriction apply when **editing existing devices** that already have "Offline" set? (i.e., should we clear the "Offline" status on affected devices, or just prevent new selections?)
2. Are there other manufacturer/model combinations you'd eventually want status restrictions for?
3. Does this apply to **both the admin panel and the customer portal**, or just one?
4. Should the restriction also apply to **bulk status updates**, or just individual device edits?

Let us know which direction you'd like to go and answers to the above, and we'll get this scoped and scheduled.

Thanks,
Aksana
