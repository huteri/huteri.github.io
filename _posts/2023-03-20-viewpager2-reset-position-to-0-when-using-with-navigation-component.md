---
layout: post
title: ViewPager2 reset position to 0 when using with Navigation Component
date: 2023-03-20 21:24 +0800
---

If you've ever used ViewPager2 with the Navigation Component in Android, you may have encountered an issue where the ViewPager2 resets its position to 0 when coming back from the backstack. This can be a frustrating problem, especially if you're not familiar with how ViewPager2 works.
<!--more-->

The problem is caused by the way RecyclerView works. ViewPager2 is built on top of RecyclerView to provide paging functionality. When we do data updates and adjust the size of the RecyclerView dynamically, it causes the RecyclerView to reset its position to 0. This can cause issues when we use ViewPager2 with the Navigation Component in fragments.

When we use ViewPager2 with the Navigation Component in fragments, it causes the recreation of the fragment and reinitialization of the ViewPager2. If we call `notifyDataSetChanged()` or `notifyItemChanged()` on initialization of the fragment, it will reset the ViewPager2 position to 0 when the screen resumes.

To solve this issue, we need to check if we have called `notifyDataSetChanged()` before and ignore it to preserve the state of ViewPager2 `onPopBackStack()`. This will prevent the ViewPager2 from resetting its position to 0 when coming back from the backstack.

Here's an example of how to implement this solution:

```kotlin
class DetailsFragment : Fragment() {

    var hasShownProgressTab: Boolean = false

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View {
        binding = FragmentBinding.inflate(inflater, container, false)
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        initTabs()

        // delay call initViews()
    }

    private fun initTabs() {
        adapter = TabsAdapter(this)
        adapter.hasJoined = hasShownProgressTab

        binding.tabsPager.adapter = adapter

        TabLayoutMediator(binding.tabsLayout, binding.tabsPager) { tabs, position ->

            tabs.text = when (position) {
                0 -> getString(R.string.txt_details)
                1 -> getString(R.string.txt_progress)
                2 -> getString(R.string.txt_leaderboard)
                else -> null
            }
        }.attach()

    }


    private fun initViews() {

        if (teamChallenge?.teamId == null) {
            hasShownProgressTab = false
            adapter.hideProgressTab()
        } else {
            hasShownProgressTab = true
            adapter.showProgressTab()
        }
    }

}

class TabsAdapter(fragment: Fragment) : FragmentStateAdapter(fragment) {

    var hasJoined: Boolean = false
    private val detailsTabFragment by lazy { DetailsTabFragment() }
    private val progressTabFragment by lazy { ProgressTabFragment() }
    private val leaderboardTabFragment by lazy { LeaderboardTabFragment() }

    override fun getItemCount(): Int {
        return if (hasJoined) 3 else 1
    }

    override fun createFragment(position: Int): Fragment {
        return when (position) {
            0 -> detailsTabFragment
            1 -> progressTabFragment
            else -> leaderboardTabFragment
        }
    }

    fun showProgressTab() {
        if (hasJoined) return

        hasJoined = true

        notifyItemRangeInserted(1, 2)
    }

    fun hideProgressTab() {
        if (!hasJoined) return

        hasJoined = false

        notifyItemRangeRemoved(1, 2)
    }
}

```