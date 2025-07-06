---
layout: post
title: Become a Pandas Wizard Overnight
subtitle: You've been doing Pandas wrong all this time.
tags: [data]
---

If you are a data analyst, chances are that there would be portions of your work that you would do in pandas. And once you've been in it for a while, it sort of becomes second nature writing in pandas, as it's pretty easy to build the muscle memory once you have been exposed enough to it.

Pandas is very easy to use; almost all of the functionalities there can be written with one-liners. And with that easiness, it's also easy to write in such an unconventional manner.

There would come a time where **your messy pandas code would haunt you**, as it definitely happened to me. It specifically goes like this: I was creating an EDA to better understand our customers who are using buy now pay later (BNPL). As the EDA is more like an iterative process, each time I stepped away from it for a while and came back, it felt like there was a lot of friction in understanding my previous work. There also came a time when I handed it to our data scientist, and it definitely made it hard for her to read my code.

Additionally, I think this is the worst situation you would encounter: if suddenly you are in a meeting presenting your progress and then a question is asked, perhaps questioning your methodologies or asking for the specific value(s) that you got in your analysis, the answer is definitely in the code, but your code is so messy that it would take you a while to locate the answer.

Going back, this is more of a rant rather than a tutorial; I have nothing to teach but something to preach. 🫳🎤 

If you want to become a pandas wizard overnight, I'll give you this tip:

> <mark>Use Method Chains</mark>

No more, no less. If there is only one piece of advice that I can give to beginners in pandas, it would be this. For example:

```python
monthly_customer_behavior_df = paid_df.groupby(by=['customer_id', 'year_month'], as_index=False).agg(
    # Transactions
    no_txns=('pay_later_id', 'nunique'),
    ontime_txn=('payment_status', lambda x: (x == 'ontime').sum()),
    late_txn=('payment_status', lambda x: (x == 'late').sum()),

    # Payment Behavior
    sum_days_early=('days_before_paid', 'sum'),
    avg_days_early=('days_before_paid', 'mean'),
    mdn_days_early=('days_before_paid', 'median'),
    days_early_list=('days_before_paid', lambda x: x.tolist()),
    std_days_early=('days_before_paid', 'std'),

    # Credit
    avg_credit_limit=('credit_limit', 'mean'),
    avg_utilization=('credit_utilization', 'mean'),
)

monthly_customer_behavior_df["payment_behavior"] = monthly_customer_behavior_df["std_days_early"].apply(lambda x: "STABLE" if x < 2.5 else "VOLATILE")

monthly_customer_behavior_df.sort_values(['customer_id', 'year_month'], inplace=True)

monthly_customer_behavior_df
```

This is how people often write in pandas. You can imagine that these may be scattered across different cells in a Jupyter notebook or similar environments. To be honest, this is what most people consider clean pandas. They scan the entire notebook and delete isolated code snippets used mainly for testing and verifying data transformations. In the end, all that remains is a notebook containing various analyses. An improved version of the example above would be like this:

```python
monthly_customer_behavior_df = (
    paid_df
        .groupby(by=['customer_id', 'year_month'], as_index=False)
        .agg(
            # Transactions
            no_txns=('pay_later_id', 'nunique'),
            ontime_txn=('payment_status', lambda x: (x == 'ontime').sum()),
            late_txn=('payment_status', lambda x: (x == 'late').sum()),

            # Payment Behavior
            sum_days_early=('days_before_paid', 'sum'),
            avg_days_early=('days_before_paid', 'mean'),
            mdn_days_early=('days_before_paid', 'median'),
            std_days_early=('days_before_paid', 'std'),

            # Credit
            avg_credit_limit=('credit_limit', 'mean'),
            avg_utilization=('credit_utilization', 'mean'),
        )
        .assign(payment_behavior = lambda x: x['std_days_early'].apply(lambda x: "STABLE" if x < 2.5 else "VOLATILE"))
        .sort_values(by=['customer_id', 'year_month'])
)

monthly_customer_behavior_df
```

Notice that all of the transformation are all contained in a single declaration where the chunk of code represents various transformations that produces a single output. Here is the visual representation of what I am trying to convey:

<div align="center">
  <img src="/assets/images/from_posts/2025-07-06-become-a-pandas-wizard.svg" alt="">
</div>

While the difference might seem negligible in this example, its benefit becomes apparent when dealing with a large number of data transformations. 

The formatting of method chains is a matter of personal preference. Personally, I indent the method and create an additional indent for the parameters and their arguments:

```python
monthly_customer_behavior_df = (
    paid_df
        .groupby(
            by=['customer_id', 'year_month'], as_index=False
        )
        .agg(
            # Transactions
            no_txns=('pay_later_id', 'nunique'),
            ontime_txn=('payment_status', lambda x: (x == 'ontime').sum()),
            late_txn=('payment_status', lambda x: (x == 'late').sum()),

            # Payment Behavior
            sum_days_early=('days_before_paid', 'sum'),
            avg_days_early=('days_before_paid', 'mean'),
            mdn_days_early=('days_before_paid', 'median'),
            std_days_early=('days_before_paid', 'std'),

            # Credit
            avg_credit_limit=('credit_limit', 'mean'),
            avg_utilization=('credit_utilization', 'mean'),
        )
        .assign(
            payment_behavior = lambda x: x['std_days_early'].apply(
                lambda x: "STABLE" if x < 2.5 else "VOLATILE"
            )
        )
        .sort_values(
            by=['customer_id', 'year_month']
        )
)

monthly_customer_behavior_df
```

Now, the code is pretty much self-documenting. The use of comments can be redirected towards code organization or providing the business decision behind a certain data transformation. Once you learn this and start applying it, you would gravitate on it, perhaps you could ask adjacent questions like:

- Can I method chain on custom functions? Yes, use `pandas.DataFrame.pipe`.
- Can I method chain on column creations? Yes, use `pandas.DataFrame.assign`.
- Can I method chain on filtering? Yes, use `pandas.DataFrame.query`.

**Everything in pandas can be chained.**

<div class="conclusion-divider">
    <hr>
</div>

If you are using pandas on a daily you may already know this, but this writing is best absorbed for practitioners who only use pandas occasionally. I didn't read Matt Harrison's books, but this summarizes his main idea, as stated himself at the end of this video. If you want some related tips to complement this, you can look here; these are the best that I found prior to this writing:
Making Pandas Code Neat and Maintainable

- [Making Pandas Code Neat and Maintainable](https://medium.com/henkel-data-and-analytics/making-pandas-code-neat-and-maintainable-e5920f7d2d36)
- [The ultimate reference for clean Pandas code](https://towardsdatascience.com/the-ultimate-reference-for-clean-pandas-code-413df676e63c)

PS: I could still remember the day when I first refactored my notebook to use method chains, February 21, 2025.