package lab4_204_12.uwaterloo.ca.lab4_204_12;

import android.graphics.PointF;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import mapper.LineSegment;
import mapper.MapView;
import mapper.NavigationalMap;
import mapper.VectorUtils;

public class Pathfinding{
    private MapView mapView;
    private Map<ScoredPoint, List<ScoredPoint>> mapGraph;

    private Pathfinding(MapView mapView, Map<ScoredPoint, List<ScoredPoint>> mapGraph){
        this.mapView = mapView;
        this.mapGraph = mapGraph;
    }

    public static Pathfinding preProcessMap(MapView mapView){

        NavigationalMap map = mapView.getMap();

        //Find all vertices on the map
        List<ScoredPoint> vertices = new ArrayList<>();
        List<LineSegment> lines = map.getGeometry();

        boolean linesShareVertices;
        float[] innerUnitVector;
        float[] outerUnitVector;
        float scale = 0.05f;

        for(LineSegment outerLine : lines){
            for(LineSegment innerLine : lines){
                if(innerLine != outerLine){
                    PointF vertex = new PointF();
                    innerUnitVector = innerLine.findUnitVector();
                    outerUnitVector = outerLine.findUnitVector();
                    linesShareVertices = false;

                    if(innerLine.start.equals(outerLine.start)){
                        linesShareVertices = true;
                        vertex.set(innerLine.start);
                        vertex.offset(scale*(-innerUnitVector[0]-outerUnitVector[0]),scale*(-innerUnitVector[1]-outerUnitVector[1]));
                    }else if(innerLine.end.equals(outerLine.start)){
                        linesShareVertices = true;
                        vertex.set(innerLine.end);
                        vertex.offset(scale*(innerUnitVector[0]-outerUnitVector[0]),scale*(innerUnitVector[1]-outerUnitVector[1]));
                    }else if(innerLine.start.equals(outerLine.end)){
                        linesShareVertices = true;
                        vertex.set(innerLine.start);
                        vertex.offset(scale*(-innerUnitVector[0]+outerUnitVector[0]),scale*(-innerUnitVector[1]+outerUnitVector[1]));
                    }else if(innerLine.end.equals(outerLine.end)){
                        linesShareVertices = true;
                        vertex.set(innerLine.end);
                        vertex.offset(scale*(innerUnitVector[0]+outerUnitVector[0]),scale*(innerUnitVector[1]+outerUnitVector[1]));
                    }

                    if(linesShareVertices && !vertices.contains(vertex)){
                        vertices.add(new ScoredPoint(vertex));
                    }
                }
            }
        }

        Map<ScoredPoint, List<ScoredPoint>> mapGraph = new HashMap<>(vertices.size());
        for(ScoredPoint vertex:vertices){
            ArrayList<ScoredPoint> visibleVertices = new ArrayList<>();

            for(ScoredPoint candidateVertex:vertices){
                if(candidateVertex != vertex){
                    int nIntersections = map.calculateIntersections(candidateVertex.coords, vertex.coords).size();
                    if(nIntersections==0){
                        visibleVertices.add(candidateVertex);
                    }
                }
            }

            mapGraph.put(vertex, visibleVertices);
        }

        return new Pathfinding(mapView, mapGraph);
    }

    public List<PointF> calculatePath(){
        List<PointF> path = new ArrayList<PointF>();
        NavigationalMap map = mapView.getMap();
        ScoredPoint start = new ScoredPoint(mapView.getUserPoint());
        start.score = 0;
        ScoredPoint end = new ScoredPoint(mapView.getDestinationPoint());

        if((start.coords.x==0 && start.coords.y==0) || (end.coords.x==0 && end.coords.y==0)){
            return path;
        }


        List<ScoredPoint> unvisited = new ArrayList<>(mapGraph.size()+2);
        unvisited.add(start);
        unvisited.add(end);
        for(ScoredPoint vertex:mapGraph.keySet()){
            unvisited.add(vertex);
            vertex.score = Double.POSITIVE_INFINITY;
            vertex.prev = null;
        }
    /*
        path.add(start.coords);
        for(ScoredPoint vertex:getVisibleNodes(start, unvisited, end)){
            path.add(vertex.coords);
            path.add(start.coords);
        }

        */
        ScoredPoint currentPoint = start;
        while(currentPoint != end){
            List<ScoredPoint> visiblePoints = getVisibleNodes(currentPoint, unvisited, end);
            for(ScoredPoint node : visiblePoints){
                double score = currentPoint.score + VectorUtils.distance(currentPoint.coords, node.coords);
                if(score < node.score){
                    node.score = score;
                    node.prev = currentPoint;
                }
            }
            unvisited.remove(currentPoint);
            currentPoint = getClosestPoint(unvisited);
            if(currentPoint.score==Double.POSITIVE_INFINITY){
                System.out.println("You broke it...");
                return new ArrayList<>();
            }
        }

        while(currentPoint!=null && currentPoint!=start){
            path.add(currentPoint.coords);
            currentPoint = currentPoint.prev;
        }
        path.add(currentPoint.coords);
        //*/
        return path;
    }

    private static ScoredPoint getClosestPoint(List<ScoredPoint> list){
        if(list.size()==0)
            throw new IllegalStateException("List is empty");
        ScoredPoint closest = list.get(0);
        double lowestScore = Double.POSITIVE_INFINITY;
        for(ScoredPoint point:list){
            if(point.score<lowestScore){
                lowestScore = point.score;
                closest = point;
            }
        }
        return closest;
    }

    private List<ScoredPoint> getVisibleNodes(ScoredPoint currentNode, List<ScoredPoint> unvisited, ScoredPoint end){
        NavigationalMap map = mapView.getMap();
        List<ScoredPoint> visibleNodes = mapGraph.get(currentNode);
        if(visibleNodes!=null){
            if(map.calculateIntersections(currentNode.coords, end.coords).size()==0){
                visibleNodes.add(end);
            }
            return visibleNodes;
        }
        visibleNodes = new ArrayList<>();

        for(ScoredPoint vertex:unvisited){
            if(vertex != currentNode){
                int nIntersections = map.calculateIntersections(currentNode.coords, vertex.coords).size();
                if(nIntersections==0){
                    visibleNodes.add(vertex);
                }
            }
        }

        return visibleNodes;
    }

    private static class ScoredPoint{
        private double score;
        private PointF coords;
        private ScoredPoint prev;

        public ScoredPoint(PointF coords){
            this.coords = coords;
            score = Double.POSITIVE_INFINITY;
            prev = null;
        }
    }
}
