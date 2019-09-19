Determining the Logic Result of a Subroutine It is worth noting again that the configured actions within a subroutine do not in any way affect whether a particular route is advertised by the router. The subroutine actions are used only to determine the true or false result. To illustrate this point, assume that main-policy is applied as we saw in the “Policy Subroutines” section. In this instance, however, the policies are altered as so:

```
[edit policy-options]
user@Merlot# show policy-statement main-policy
term subroutine-as-a-match {
  from policy subroutine-policy;
  then accept;
}

[edit policy-options]
user@Merlot# show policy-statement subroutine-policy
term get-routes {
  from protocol static;
  then accept;
}
term no-BGP-routes {
  from protocol bgp;
  then reject;
}
```

We are now aware of the protocol default policy being evaluated within the subroutine, so subroutine-policy now has an explicit term rejecting all BGP routes. Because they are rejected within the subroutine, there is no need within main-policy for an explicit then reject term. You may already see the flaw in this configuration, but let’s follow the logic. The router evaluates the first term of main-policy and finds a match criterion of from policy subroutine-policy. It then evaluates the first term of the subroutine and finds that all static routes have an action of then accept. This returns a true result to main-policy, where the subroutine-as-a-match term has a configured action of then accept. The static routes are now truly accepted and are advertised to the EBGP peer.

When it comes to the BGP routes in the routing table, things occur a bit differently. When the router enters the subroutine, it finds the no-BGP-routes term where all BGP routes are rejected. This returns a false result to main-policy, which means that the criterion in the subroutine-asa- match term doesn’t match. This causes the routes to move to the next configured term in main olicy, which has no other terms. The router then evaluates the next policy in the policy chain— the BGP default policy. The default policy, of course, accepts all BGP routes, and they are advertised to the EBGP peer.
