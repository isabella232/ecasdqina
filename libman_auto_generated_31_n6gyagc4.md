---
layout: lib
permalink: geometory/geometory
title: 基本

---

no verify

// @ Geometory Library

```cpp
/// --- Geometory Circle Library {{"{{"}}{ ///
// center, radius
using Circle = pair< Point, Scalar >;

// -1 : 0 share (outside)
//  1 : 0 share (B in A)
//  2 : 0 share (A in B)
// -3 : 1 share (outside)
//  3 : 1 share (B in A)
//  4 : 1 share (A in B)
//  0 : 2 share
int cpr(Circle a, Circle b) {
  Scalar d = norm(a.first - b.first);
  if(a.second + b.second + EPS < d) return -1;
  if(b.second + d + EPS < a.second) return 1;
  if(a.second + d + EPS < b.second) return 2;
  if(abs(a.second + b.second - d) < EPS) return -3;
  if(abs(b.second + d - a.second) < EPS) return 3;
  if(abs(a.second + d - b.second) < EPS) return 4;
  return 0;
}

vector< Point > intersections(Circle a, Circle b) {
  vector< Point > res;
  // normalize(b-a) * R_A
  Point x = a.second * normalize(b.first - a.first);
  if(abs(cpr(a, b)) >= 3) {
    res.emplace_back(a.first + x);
  } else if(cpr(a, b) == 0) {
    Scalar s = arg(b.second, norm(b.first - a.first), a.second);
    res.emplace_back(a.first + x * Point(cos(s), sin(s)));
    res.emplace_back(a.first + x * Point(cos(-s), sin(-s)));
  }
  return res;
}

vector< Point > intersections(Circle a, Line line) {
  vector< Point > res;
  Point n = normal(line.first - line.second);
  Point p = intersection(line, Line(a.first, a.first + n));
  Scalar d = norm(a.first - p);
  if(abs(d - a.second) < EPS) {
    res.emplace_back(p);
  } else if(abs(d) < a.second) {
    Scalar len = sqrt(a.second * a.second - d * d);
    Point share = len * normalize(line.first - line.second);
    res.emplace_back(p + share);
    res.emplace_back(p - share);
  }
  return res;
}

inline Scalar area(Circle a) { return PI * a.second * a.second; }

Scalar shareArea(Circle a, Circle b) {
  Scalar d = norm(a.first - b.first);
  if(a.second + b.second < d + EPS) return 0;
  if(a.second < b.second) swap(a, b);
  if(b.second + d < a.second + EPS) return area(b);
  Scalar s1 = arg(b.second, a.second, d), s2 = arg(a.second, b.second, d);
  Scalar tri2 =
      (a.second * a.second * sin(s1 * 2) + b.second * b.second * sin(s2 * 2)) /
      2;
  return a.second * a.second * s1 + b.second * b.second * s2 - tri2;
}

inline Line ajacentLine(Circle c, Point p) {
  return Line(p, p + normal(p - c.first));
}

// the tangentLine of c passing p
vector< Line > tangentLine(Circle c, Point p) {
  vector< Line > res;
  Scalar d = norm(p - c.first);
  if(abs(d - c.second) < EPS)
    res.emplace_back(ajacentLine(c, p));
  else if(c.second < d) {
    Point a = c.first + c.second * normalize(p - c.first);
    vector< Point > b = intersections(Circle(c.first, norm(c.first - p)),
                                      Line(a, a + normal(c.first - p)));
    for(size_t i = 0; i < b.size(); i++) {
      res.emplace_back(p, c.first + c.second * normalize(b[i] - c.first));
    }
  }
  return res;
}

vector< Line > commonTangengLine(Circle a, Circle b) {
  vector< Line > res;
  if(a.second + EPS < b.second) swap(a, b);
  if(norm(a.first - b.first) < EPS) return res;

  Point p = a.first + (b.first - a.first) * a.second / (a.second + b.second);
  if(norm(a.first - p) + EPS > a.second) res = tangentLine(a, p);
  if(abs(a.second - b.second) < EPS) {
    Point n = normal(normalize(b.first - a.first) * a.second);
    res.emplace_back(a.first + n, b.first + n);
    res.emplace_back(a.first - n, b.first - n);
  } else {
    Point q = a.first + (b.first - a.first) * a.second / (a.second - b.second);
    if(abs(a.first - q) + EPS > a.second) {
      vector< Line > tmp = tangentLine(a, q);
      res.insert(begin(res), begin(tmp), end(tmp));
    }
  }
  return res;
}

/// }}}--- ///
```


```cpp
/// --- Geometory Polygon Library {{"{{"}}{ ///
// ok for either ccw or cw
Scalar area(Polygon poly) {
  if(poly.size() < 3) return 0;
  Scalar res = cross(poly[poly.size() - 1], poly[0]);
  for(size_t i = 0; i < poly.size() - 1; i++) {
    res += cross(poly[i], poly[i + 1]);
  }
  return abs(res) / 2;
}

//  0 : outside
//  1 : inside
// -1 : on vertex
// -2 : on line
int inside(Polygon poly, Point p) {
  int cnt = 0;
  for(size_t i = 0; i < poly.size(); i++) {
    size_t ii = i, jj = (i + 1) % poly.size();
    if(norm(poly[i] - p) < EPS) return -1;
    if(poly[ii].Y > poly[jj].Y) swap(ii, jj);
    if(poly[ii].Y - EPS < p.Y && p.Y < poly[jj].Y + EPS) {
      if(abs(poly[ii].Y - poly[jj].Y) < EPS) { // parallel
        if(poly[ii].X > poly[jj].X) swap(ii, jj);
        if(poly[ii].X - EPS < p.X && p.X < poly[jj].X + EPS) return -2;
      } else {
        Point q =
            intersection(Line(poly[ii], poly[jj]), Line(p, p + Point(1, 0)));
        if(p.X < q.X && p.Y > poly[ii].Y + EPS) cnt++; // count only upside
      }
    }
  }
  return cnt & 1;
}

// param ccwConvex must be ccw and convex
Scalar caliper(Polygon ccwConvex) {
  constexpr auto comp = [](Point a, Point b) {
    return a.X == b.X ? a.Y < b.Y : a.X < b.X;
  };
  size_t i, j;
  for(size_t k = 0; k < ccwConvex.size(); k++) {
    if(comp(ccwConvex[i], ccwConvex[k])) j = k;
    if(!comp(ccwConvex[j], ccwConvex[k])) i = k;
  }
  Scalar res = 0;
  size_t si = i, sj = j;
  while(i != si || j != sj) {
    res = max(res, norm(ccwConvex[i] - ccwConvex[j]));
    if(cross(ccwConvex[(i + 1) % ccwConvex.size()] - ccwConvex[i],
             ccwConvex[(j + 1) % ccwConvex.size()] - ccwConvex[j]) < 0)
      i = (i + 1) % ccwConvex.size();
    else
      j = (j + 1) % ccwConvex.size();
  }
  return res;
}
/// }}}--- ///
```

